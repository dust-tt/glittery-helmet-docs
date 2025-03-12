# Troubleshooting Guide: Common Issues and Solutions

This document provides detailed troubleshooting steps for common technical issues encountered when using GlitteryHelmet security solutions. The following guides address specific problems in depth, with diagnostic procedures and resolution steps for security engineers.

## Table of Contents

1. [False Positive Resolution in the AI Detection Engine](#false-positive-resolution-in-the-ai-detection-engine)
2. [Diagnosing Authentication Failures in Distributed Sensor Networks](#diagnosing-authentication-failures-in-distributed-sensor-networks)

---

## False Positive Resolution in the AI Detection Engine

### Symptom

The GlitteryHelmet AI Detection Engine is generating a high volume of false positive alerts for specific traffic patterns, particularly for east-west traffic in containerized environments. These false positives persist even after standard threshold adjustment.

### Diagnosis Steps

1. **Extract Detection Logs**

```json
{
  "alert_type": "anomaly",
  "confidence": 0.85,
  "traffic_pattern": "east-west",
  "container_context": true
}
```

2. **Analyze Alert Distribution**

   - Monitor distribution patterns
   - Track temporal variations
   - Identify cluster correlations

3. **Investigate Behavioral Models**

   - Review model parameters
   - Check training data sources
   - Validate detection thresholds

4. **Inspect Feature Importance**
   Review the feature importance scores from the model analysis. Look for features with unexpectedly high influence:
   - Network entropy: 45%
   - Protocol distribution: 30%
   - Time-based patterns: 25%

### Root Cause

In most cases, the false positives are caused by one of three issues:

1. **Training Data Bias**: The behavioral model has been trained with insufficient or non-representative data for container orchestration patterns.
2. **Feature Miscalibration**: Default entropy calculations for containerized workloads are often misconfigured for orchestration platforms like Kubernetes.
3. **Detection Rule Conflict**: Custom detection rules are conflicting with the behavioral model's baseline assumptions.

### Resolution

#### Option 1: Model Retraining with Extended Baselining

```bash
CONTAINER_BEHAVIOR_MODEL --retrain --extended-baseline --min-samples 1000
```

#### Option 2: Feature Weight Adjustment

```json
{
  "feature_weights": {
    "network_entropy": 0.3,
    "protocol_distribution": 0.4,
    "temporal_patterns": 0.3
  }
}
```

#### Option 3: Implement Custom Exclusion Rules

```yaml
exclusions:
  - pattern: "k8s-orchestration"
    confidence_threshold: 0.95
    review_period: 7d
```

### Validation

Monitor false positive rates after implementing the solution:

- Previous false positive rate: 12%
- Target false positive rate: < 5%
- Validation period: 7 days

---

## Diagnosing Authentication Failures in Distributed Sensor Networks

### Symptom

GlitteryHelmet distributed sensors are failing to authenticate with the central security platform, resulting in gaps in security coverage. Authentication logs show sporadic TLS handshake failures and certificate validation errors.

### Environmental Context

This issue typically appears in:

- Multi-site deployments with 50+ sensors
- Environments with restrictive network policies
- Systems using custom certificate authorities
- Mixed cloud/on-premises deployments

### Diagnostic Steps

1. **Verify Certificate Status**

```bash
openssl x509 -in sensor.crt -text -noout | grep "Not After"
```

2. **Test Network Connectivity**

```bash
netcat -vz control-plane.security.local 443
```

3. **Analyze Authentication Flow**
   Enable debug logging for authentication attempts:

```bash
"auth_debug=true sensor_id=sensor-01 level=debug msg='TLS handshake initiated'"
```

4. **Examine Certificate Chain**

```bash
"openssl verify -verbose -CAfile ca-bundle.crt sensor.crt"
```

5. **Investigate Clock Synchronization**

```bash
"chronyc tracking"
```

### Common Root Causes

1. **TLS Cipher Mismatch**

   - The sensor and control plane have incompatible TLS cipher configurations.

2. **Certificate Trust Chain Issues**

   - Missing intermediate certificates in the sensor trust store.

3. **Clock Drift**

   - Sensors with clock drift beyond the tolerance threshold (default: 60 seconds).

4. **Network Interception**
   - Corporate proxies or security devices performing TLS inspection.

### Resolution

#### Solution 1: Resolve TLS Cipher Mismatch

```bash
auth.ciphers='TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256'
```

#### Solution 2: Complete Certificate Chain

```bash
openssl verify -verbose -show-chain sensor.crt
```

#### Solution 3: Fix Clock Synchronization

```bash
chronyc makestep
```

#### Solution 4: Handle Network Interception

```bash
systemctl restart glitteryhelmet-sensor
```

### Validation

Perform a comprehensive authentication test:

```bash
sensor_status --check last_seen
```

Successful authentication will result in:

```bash
last_seen: 2024-03-12T14:37:45Z
```

### Preventative Measures

- Set up certificate expiration monitoring with 30-day advanced notification
- Implement automated NTP compliance checks in your monitoring system
- Include TLS configuration in your change management process
- Document corporate network security devices that perform TLS inspection
