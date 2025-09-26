# Deployment Rollbacks

## 🎯 Rollback Concepts

### What is Rollback?
- Reverting to a previous deployment version
- Quick recovery from failed updates
- Uses deployment revision history
- Maintains application availability

### When to Rollback?
- Application crashes after update
- Performance degradation
- Configuration errors
- Failed health checks

## 📚 Revision History

### View Rollout History
```bash
# Show deployment history
kubectl rollout history deployment/nginx-deployment

# Detailed history with change causes
kubectl rollout history deployment/nginx-deployment --revision=2

# Show all revisions
kubectl rollout history deployment/nginx-deployment --revision=0
```

### History Configuration
```yaml
spec:
  revisionHistoryLimit: 10  # Keep last 10 revisions (default)
```

## ⏪ Performing Rollbacks

### Step-by-Step Rollback Process

#### Step 1: Check Current Status
```bash
# Check current deployment status
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment

# Check current pods
kubectl get pods -l app=nginx
```

#### Step 2: View Rollout History
```bash
# Show deployment history
kubectl rollout history deployment/nginx-deployment

# Check specific revision details
kubectl rollout history deployment/nginx-deployment --revision=2
```

#### Step 3: Perform Rollback

**Option A: Rollback to Previous Version**
```bash
# Rollback to immediately previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback with record for audit trail
kubectl rollout undo deployment/nginx-deployment --record
```

**Option B: Rollback to Specific Revision**
```bash
# Rollback to specific revision number
kubectl rollout undo deployment/nginx-deployment --to-revision=3

# Check what revision you're rolling back to first
kubectl rollout history deployment/nginx-deployment --revision=3
```

#### Step 4: Monitor Rollback Progress
```bash
# Watch rollback progress
kubectl rollout status deployment/nginx-deployment

# Monitor pods during rollback
kubectl get pods -l app=nginx -w

# Check rollback events
kubectl describe deployment nginx-deployment
```

#### Step 5: Verify Rollback Success
```bash
# Check deployment status
kubectl get deployment nginx-deployment

# Verify image version
kubectl describe deployment nginx-deployment | grep Image

# Check pod status
kubectl get pods -l app=nginx

# Test application functionality
kubectl port-forward deployment/nginx-deployment 8080:80
curl localhost:8080
```

### Complete Rollback Example
```bash
#!/bin/bash
# rollback-deployment.sh

DEPLOYMENT_NAME="nginx-deployment"
NAMESPACE="default"

echo "Starting rollback for $DEPLOYMENT_NAME..."

# Step 1: Check current status
echo "Current deployment status:"
kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE

# Step 2: Show history
echo "Deployment history:"
kubectl rollout history deployment/$DEPLOYMENT_NAME -n $NAMESPACE

# Step 3: Perform rollback
echo "Performing rollback..."
kubectl rollout undo deployment/$DEPLOYMENT_NAME -n $NAMESPACE --record

# Step 4: Wait for rollback to complete
echo "Waiting for rollback to complete..."
kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE --timeout=300s

# Step 5: Verify rollback
if [ $? -eq 0 ]; then
    echo "✓ Rollback completed successfully"
    echo "New deployment status:"
    kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE
    kubectl describe deployment $DEPLOYMENT_NAME -n $NAMESPACE | grep Image
else
    echo "✗ Rollback failed"
    exit 1
fi
```

## 🔍 Rollback Verification

### Comprehensive Verification Steps

#### 1. Check Deployment Status
```bash
# Verify deployment is healthy
kubectl get deployment nginx-deployment
kubectl describe deployment nginx-deployment

# Check current revision number
kubectl get deployment nginx-deployment -o yaml | grep revision
```

#### 2. Verify Image Version
```bash
# Check container image
kubectl describe deployment nginx-deployment | grep Image

# Get detailed container info
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
```

#### 3. Test Application Functionality
```bash
# Port forward to test locally
kubectl port-forward deployment/nginx-deployment 8080:80 &
PORT_FORWARD_PID=$!

# Test application
curl -f localhost:8080 || echo "Application test failed"

# Clean up port forward
kill $PORT_FORWARD_PID
```

#### 4. Verify Pod Health
```bash
# Check all pods are running
kubectl get pods -l app=nginx

# Check pod readiness
kubectl get pods -l app=nginx -o custom-columns=NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status

# Check for any pod issues
kubectl describe pods -l app=nginx | grep -A 5 Events
```

#### 5. Compare Revisions
```bash
# Compare current with previous revision
kubectl rollout history deployment/nginx-deployment --revision=1
kubectl rollout history deployment/nginx-deployment --revision=2

# Get full deployment specification
kubectl get deployment nginx-deployment -o yaml > current-deployment.yaml
```

### Automated Verification Script
```bash
#!/bin/bash
# verify-rollback.sh

DEPLOYMENT_NAME="nginx-deployment"
NAMESPACE="default"
EXPECTED_IMAGE="nginx:1.19"  # Expected image after rollback

echo "Verifying rollback for $DEPLOYMENT_NAME..."

# Check deployment status
STATUS=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.status.conditions[?(@.type=="Progressing")].status}')
if [ "$STATUS" != "True" ]; then
    echo "✗ Deployment is not progressing correctly"
    exit 1
fi

# Check image version
CURRENT_IMAGE=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')
if [ "$CURRENT_IMAGE" = "$EXPECTED_IMAGE" ]; then
    echo "✓ Image version correct: $CURRENT_IMAGE"
else
    echo "✗ Image version incorrect. Expected: $EXPECTED_IMAGE, Got: $CURRENT_IMAGE"
    exit 1
fi

# Check pod readiness
READY_PODS=$(kubectl get pods -l app=nginx -n $NAMESPACE --no-headers | grep Running | wc -l)
DESIRED_PODS=$(kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.replicas}')

if [ "$READY_PODS" -eq "$DESIRED_PODS" ]; then
    echo "✓ All $READY_PODS pods are ready"
else
    echo "✗ Only $READY_PODS/$DESIRED_PODS pods are ready"
    exit 1
fi

# Test application
echo "Testing application functionality..."
kubectl port-forward deployment/$DEPLOYMENT_NAME 8080:80 -n $NAMESPACE &
PORT_FORWARD_PID=$!
sleep 5

if curl -f -s localhost:8080 > /dev/null; then
    echo "✓ Application is responding correctly"
else
    echo "✗ Application is not responding"
    kill $PORT_FORWARD_PID
    exit 1
fi

kill $PORT_FORWARD_PID
echo "✓ Rollback verification completed successfully"
```

## 🚨 Rollback Troubleshooting

### Failed Rollbacks
```bash
# Check rollback status
kubectl rollout status deployment/nginx-deployment

# Check deployment conditions
kubectl describe deployment nginx-deployment | grep -A 5 Conditions

# Check ReplicaSet issues
kubectl get rs -l app=nginx
kubectl describe rs <old-replicaset>
```

### Common Rollback Issues
- Insufficient resources for old version
- Old image no longer available
- Configuration conflicts
- Persistent volume issues

### Debug Commands
```bash
# Check deployment events
kubectl get events --field-selector involvedObject.name=nginx-deployment

# Check pod status after rollback
kubectl get pods -l app=nginx
kubectl describe pod <pod-name>

# Verify service endpoints
kubectl get endpoints <service-name>
```

## 📋 Rollback Strategies

### 1. Immediate Rollback (Emergency)
**Use Case**: Critical production issues requiring immediate action

```bash
# Quick rollback for critical issues
echo "EMERGENCY ROLLBACK INITIATED"
kubectl rollout undo deployment/nginx-deployment --record

# Monitor rollback progress
kubectl rollout status deployment/nginx-deployment --timeout=300s

# Verify immediately
kubectl get pods -l app=nginx
echo "Emergency rollback completed"
```

### 2. Planned Rollback (Controlled)
**Use Case**: Scheduled rollback with proper verification

```bash
#!/bin/bash
# planned-rollback.sh

DEPLOYMENT="nginx-deployment"
TARGET_REVISION="2"

echo "=== PLANNED ROLLBACK PROCEDURE ==="

# Step 1: Identify target revision
echo "1. Current deployment history:"
kubectl rollout history deployment/$DEPLOYMENT

# Step 2: Review target revision details
echo "2. Target revision details:"
kubectl rollout history deployment/$DEPLOYMENT --revision=$TARGET_REVISION

# Step 3: Confirm rollback
read -p "Proceed with rollback to revision $TARGET_REVISION? (y/N): " confirm
if [ "$confirm" != "y" ]; then
    echo "Rollback cancelled"
    exit 0
fi

# Step 4: Perform rollback
echo "3. Performing rollback..."
kubectl rollout undo deployment/$DEPLOYMENT --to-revision=$TARGET_REVISION --record

# Step 5: Monitor progress
echo "4. Monitoring rollback progress..."
kubectl rollout status deployment/$DEPLOYMENT --timeout=600s

# Step 6: Verify rollback
echo "5. Verifying rollback..."
kubectl get deployment $DEPLOYMENT
kubectl describe deployment $DEPLOYMENT | grep Image

echo "Planned rollback completed successfully"
```

### 3. Canary Rollback (Gradual)
**Use Case**: Test rollback with minimal risk

```bash
#!/bin/bash
# canary-rollback.sh

DEPLOYMENT="nginx-deployment"
CANARY_NAME="nginx-canary"
OLD_IMAGE="nginx:1.19"

echo "=== CANARY ROLLBACK PROCEDURE ==="

# Step 1: Scale down current deployment
echo "1. Scaling down current deployment..."
ORIGINAL_REPLICAS=$(kubectl get deployment $DEPLOYMENT -o jsonpath='{.spec.replicas}')
kubectl scale deployment $DEPLOYMENT --replicas=1

# Step 2: Create canary with old version
echo "2. Creating canary deployment..."
kubectl create deployment $CANARY_NAME --image=$OLD_IMAGE
kubectl scale deployment $CANARY_NAME --replicas=1

# Step 3: Wait for canary to be ready
echo "3. Waiting for canary to be ready..."
kubectl wait --for=condition=available --timeout=300s deployment/$CANARY_NAME

# Step 4: Test canary
echo "4. Testing canary deployment..."
kubectl port-forward deployment/$CANARY_NAME 8081:80 &
CANARY_PID=$!
sleep 5

if curl -f -s localhost:8081 > /dev/null; then
    echo "✓ Canary test successful"
    CANARY_SUCCESS=true
else
    echo "✗ Canary test failed"
    CANARY_SUCCESS=false
fi

kill $CANARY_PID

# Step 5: Decision point
if [ "$CANARY_SUCCESS" = true ]; then
    echo "5. Canary successful, proceeding with full rollback..."
    kubectl rollout undo deployment/$DEPLOYMENT --record
    kubectl rollout status deployment/$DEPLOYMENT
    kubectl scale deployment $DEPLOYMENT --replicas=$ORIGINAL_REPLICAS
    echo "✓ Full rollback completed"
else
    echo "5. Canary failed, restoring original state..."
    kubectl scale deployment $DEPLOYMENT --replicas=$ORIGINAL_REPLICAS
    echo "✗ Rollback aborted, original deployment restored"
fi

# Cleanup canary
echo "6. Cleaning up canary deployment..."
kubectl delete deployment $CANARY_NAME

echo "Canary rollback procedure completed"
```

### 4. Blue-Green Rollback
**Use Case**: Instant rollback with zero downtime

```bash
#!/bin/bash
# blue-green-rollback.sh

BLUE_DEPLOYMENT="nginx-blue"
GREEN_DEPLOYMENT="nginx-green"
SERVICE_NAME="nginx-service"
OLD_IMAGE="nginx:1.19"

echo "=== BLUE-GREEN ROLLBACK PROCEDURE ==="

# Step 1: Create green deployment with old version
echo "1. Creating green deployment with previous version..."
kubectl create deployment $GREEN_DEPLOYMENT --image=$OLD_IMAGE
kubectl scale deployment $GREEN_DEPLOYMENT --replicas=3

# Step 2: Wait for green deployment
echo "2. Waiting for green deployment to be ready..."
kubectl wait --for=condition=available --timeout=300s deployment/$GREEN_DEPLOYMENT

# Step 3: Test green deployment
echo "3. Testing green deployment..."
kubectl port-forward deployment/$GREEN_DEPLOYMENT 8082:80 &
GREEN_PID=$!
sleep 5

if curl -f -s localhost:8082 > /dev/null; then
    echo "✓ Green deployment test successful"
    GREEN_SUCCESS=true
else
    echo "✗ Green deployment test failed"
    GREEN_SUCCESS=false
fi

kill $GREEN_PID

# Step 4: Switch traffic if green is healthy
if [ "$GREEN_SUCCESS" = true ]; then
    echo "4. Switching service to green deployment..."
    kubectl patch service $SERVICE_NAME -p '{"spec":{"selector":{"app":"'$GREEN_DEPLOYMENT'"}}}'
    
    echo "5. Verifying service switch..."
    sleep 10
    
    # Test through service
    kubectl port-forward service/$SERVICE_NAME 8083:80 &
    SERVICE_PID=$!
    sleep 5
    
    if curl -f -s localhost:8083 > /dev/null; then
        echo "✓ Service switch successful, cleaning up blue deployment"
        kubectl delete deployment $BLUE_DEPLOYMENT
        echo "✓ Blue-green rollback completed"
    else
        echo "✗ Service switch failed, reverting"
        kubectl patch service $SERVICE_NAME -p '{"spec":{"selector":{"app":"'$BLUE_DEPLOYMENT'"}}}'
    fi
    
    kill $SERVICE_PID
else
    echo "4. Green deployment failed, cleaning up..."
    kubectl delete deployment $GREEN_DEPLOYMENT
    echo "✗ Blue-green rollback failed"
fi

echo "Blue-green rollback procedure completed"
```

## 🛡️ Rollback Best Practices

### Prevention
- Thorough testing before deployment
- Gradual rollout strategies
- Comprehensive monitoring
- Automated health checks

### Preparation
- Keep revision history
- Document deployment changes
- Maintain rollback procedures
- Test rollback process regularly

### Execution
- Monitor application during rollback
- Verify functionality after rollback
- Communicate rollback to team
- Investigate root cause of failure

## 📊 Rollback Automation

### Automated Rollback Script
```bash
#!/bin/bash
# automated-rollback.sh - Production-ready rollback automation

set -euo pipefail

# Configuration
DEPLOYMENT=${1:-}
NAMESPACE=${2:-default}
TARGET_REVISION=${3:-}
TIMEOUT=${4:-300}
SLACK_WEBHOOK=${SLACK_WEBHOOK:-}

# Logging function
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Slack notification function
notify_slack() {
    if [ -n "$SLACK_WEBHOOK" ]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{\"text\":\"$1\"}" \
            "$SLACK_WEBHOOK"
    fi
}

# Validation
if [ -z "$DEPLOYMENT" ]; then
    log "ERROR: Deployment name is required"
    echo "Usage: $0 <deployment-name> [namespace] [target-revision] [timeout]"
    exit 1
fi

# Check if deployment exists
if ! kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE" >/dev/null 2>&1; then
    log "ERROR: Deployment $DEPLOYMENT not found in namespace $NAMESPACE"
    exit 1
fi

log "Starting automated rollback for $DEPLOYMENT in namespace $NAMESPACE"
notify_slack "🔄 Starting rollback for $DEPLOYMENT"

# Pre-rollback checks
log "Performing pre-rollback checks..."
CURRENT_REPLICAS=$(kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE" -o jsonpath='{.spec.replicas}')
CURRENT_IMAGE=$(kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE" -o jsonpath='{.spec.template.spec.containers[0].image}')

log "Current state: $CURRENT_REPLICAS replicas, image: $CURRENT_IMAGE"

# Show rollout history
log "Deployment history:"
kubectl rollout history deployment/"$DEPLOYMENT" -n "$NAMESPACE"

# Perform rollback
if [ -n "$TARGET_REVISION" ]; then
    log "Rolling back to revision $TARGET_REVISION"
    kubectl rollout undo deployment/"$DEPLOYMENT" -n "$NAMESPACE" --to-revision="$TARGET_REVISION" --record
else
    log "Rolling back to previous revision"
    kubectl rollout undo deployment/"$DEPLOYMENT" -n "$NAMESPACE" --record
fi

# Monitor rollback progress
log "Monitoring rollback progress..."
if kubectl rollout status deployment/"$DEPLOYMENT" -n "$NAMESPACE" --timeout="${TIMEOUT}s"; then
    log "✓ Rollback completed successfully"
    
    # Post-rollback verification
    NEW_IMAGE=$(kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE" -o jsonpath='{.spec.template.spec.containers[0].image}')
    READY_REPLICAS=$(kubectl get deployment "$DEPLOYMENT" -n "$NAMESPACE" -o jsonpath='{.status.readyReplicas}')
    
    log "New state: $READY_REPLICAS/$CURRENT_REPLICAS replicas ready, image: $NEW_IMAGE"
    
    # Health check
    log "Performing post-rollback health check..."
    sleep 30  # Allow time for pods to stabilize
    
    HEALTHY_PODS=$(kubectl get pods -l app="$DEPLOYMENT" -n "$NAMESPACE" --field-selector=status.phase=Running --no-headers | wc -l)
    
    if [ "$HEALTHY_PODS" -eq "$CURRENT_REPLICAS" ]; then
        log "✓ All pods are healthy after rollback"
        notify_slack "✅ Rollback completed successfully for $DEPLOYMENT. Image: $NEW_IMAGE"
        exit 0
    else
        log "⚠ Warning: Only $HEALTHY_PODS/$CURRENT_REPLICAS pods are healthy"
        notify_slack "⚠️ Rollback completed but only $HEALTHY_PODS/$CURRENT_REPLICAS pods are healthy for $DEPLOYMENT"
        exit 1
    fi
else
    log "✗ Rollback failed or timed out"
    notify_slack "❌ Rollback failed for $DEPLOYMENT"
    
    # Show current status for debugging
    log "Current deployment status:"
    kubectl describe deployment "$DEPLOYMENT" -n "$NAMESPACE" | tail -20
    
    exit 1
fi
```

### Health-Check Based Rollback
```bash
#!/bin/bash
# health-check-rollback.sh - Rollback based on health metrics

DEPLOYMENT=$1
NAMESPACE=${2:-default}
HEALTH_ENDPOINT=${3:-"/health"}
SUCCESS_THRESHOLD=${4:-95}  # 95% success rate
CHECK_DURATION=${5:-300}    # 5 minutes

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Function to check application health
check_health() {
    local success_count=0
    local total_count=0
    local end_time=$(($(date +%s) + CHECK_DURATION))
    
    log "Starting health check for $CHECK_DURATION seconds..."
    
    while [ $(date +%s) -lt $end_time ]; do
        # Get a random pod to test
        POD=$(kubectl get pods -l app="$DEPLOYMENT" -n "$NAMESPACE" -o jsonpath='{.items[0].metadata.name}')
        
        if [ -n "$POD" ]; then
            if kubectl exec "$POD" -n "$NAMESPACE" -- curl -f -s "localhost:8080$HEALTH_ENDPOINT" >/dev/null 2>&1; then
                ((success_count++))
            fi
            ((total_count++))
        fi
        
        sleep 10
    done
    
    if [ $total_count -gt 0 ]; then
        local success_rate=$((success_count * 100 / total_count))
        log "Health check results: $success_count/$total_count successful ($success_rate%)"
        
        if [ $success_rate -lt $SUCCESS_THRESHOLD ]; then
            log "Health check failed: $success_rate% < $SUCCESS_THRESHOLD%"
            return 1
        else
            log "Health check passed: $success_rate% >= $SUCCESS_THRESHOLD%"
            return 0
        fi
    else
        log "No health checks could be performed"
        return 1
    fi
}

# Main logic
log "Monitoring deployment health: $DEPLOYMENT"

if ! check_health; then
    log "Health check failed, initiating automatic rollback..."
    
    # Perform rollback
    kubectl rollout undo deployment/"$DEPLOYMENT" -n "$NAMESPACE" --record
    
    # Wait for rollback to complete
    if kubectl rollout status deployment/"$DEPLOYMENT" -n "$NAMESPACE" --timeout=300s; then
        log "✓ Automatic rollback completed"
        
        # Verify health after rollback
        sleep 60  # Allow time for stabilization
        if check_health; then
            log "✓ Health check passed after rollback"
        else
            log "⚠ Health check still failing after rollback - manual intervention required"
        fi
    else
        log "✗ Rollback failed - manual intervention required"
    fi
else
    log "✓ Deployment is healthy, no rollback needed"
fi
```

### Kubernetes Job for Automated Rollback
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: automated-rollback-job
spec:
  template:
    spec:
      serviceAccountName: rollback-service-account
      containers:
      - name: rollback
        image: bitnami/kubectl:latest
        command:
        - /bin/bash
        - -c
        - |
          DEPLOYMENT=${DEPLOYMENT_NAME}
          NAMESPACE=${NAMESPACE:-default}
          
          echo "Starting automated rollback for $DEPLOYMENT"
          
          # Check if rollback is needed (example: check error rate)
          ERROR_RATE=$(kubectl top pods -l app=$DEPLOYMENT --no-headers | awk '{sum+=$3} END {print sum/NR}')
          
          if (( $(echo "$ERROR_RATE > 50" | bc -l) )); then
            echo "High error rate detected: $ERROR_RATE%. Initiating rollback..."
            kubectl rollout undo deployment/$DEPLOYMENT -n $NAMESPACE --record
            kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE
            echo "Rollback completed"
          else
            echo "Error rate acceptable: $ERROR_RATE%. No rollback needed"
          fi
        env:
        - name: DEPLOYMENT_NAME
          value: "nginx-deployment"
        - name: NAMESPACE
          value: "default"
      restartPolicy: Never
```