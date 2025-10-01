# **Feature/Task: Implement Kubernetes Graceful Shutdown and Health Checks**

## **Overview**

This ticket outlines the implementation of proper graceful shutdown mechanisms and Kubernetes health checks (Liveness and Readiness probes) for our API applications. Currently, applications may experience abrupt termination during deployments, scaling events, or node maintenance, leading to dropped requests and degraded user experience. By implementing these features, we aim to achieve zero-downtime deployments and improve the overall resilience and availability of our services.

## **Problem Statement**

Without proper SIGTERM handling and Kubernetes probes, our applications:

* May drop in-flight requests during pod termination.  
* Could receive traffic before they are fully initialized (e.g., database connections established).  
* Might remain in a "running" state while being unresponsive or deadlocked, leading to service degradation.

## **Solution**

Implement the following:

1. **Application-level SIGTERM handler:** To gracefully shut down the application, allowing in-flight requests to complete within a specified timeout.  
2. **/readyz API endpoint:** To signal when the application is ready to accept new traffic, including robust checks for all critical dependencies performed in parallel, handling temporary issues, and signaling unreadiness during shutdown.  
3. **/healthz API endpoint:** To signal the core liveness of the application process.  
4. **Kubernetes Deployment Configuration:** Update Deployment manifests to include livenessProbe, readinessProbe, and terminationGracePeriodSeconds.

## **Acceptance Criteria**

* **Graceful Shutdown:** During a Kubernetes-initiated pod termination (e.g., rolling update, kubectl delete pod), the application should:  
  * Stop accepting new requests immediately upon receiving SIGTERM.  
  * Continue processing all in-flight requests until completion or an **application-defined shutdown timeout of 60 seconds** is reached.  
  * Log the graceful shutdown process.  
  * Exit cleanly (process terminates with exit code 0) within the application's 60-second timeout.  
* **/readyz Endpoint:**  
  * Returns 200 OK when the application is fully initialized and **all critical external dependencies (e.g., database, Redis, message queue, essential external APIs) are available and responsive.**  
  * **Dependency checks within /readyz must be performed in parallel** to ensure efficiency and avoid blocking the health check.  
  * Returns a non-2xx status code (e.g., 503 Service Unavailable) if critical dependencies are unavailable for a configured threshold (e.g., 2-3 consecutive failures) to tolerate brief hiccups.  
  * Returns a non-2xx status code immediately upon receiving SIGTERM to signal that it should no longer receive new traffic.  
* **/healthz Endpoint:**  
  * Returns 200 OK if the core application process is running and not deadlocked.  
  * Does *not* check external dependencies that are not critical for the process itself to be "alive".  
  * Continues to return 200 OK during the terminationGracePeriodSeconds after SIGTERM is received, as long as the process is still draining requests.  
* **Kubernetes Configuration:**  
  * All relevant Deployment manifests are updated with livenessProbe, readinessProbe, and terminationGracePeriodSeconds.  
  * terminationGracePeriodSeconds is set to **75 seconds** to provide a safety margin for the application's 60-second shutdown timeout.  
  * Probe parameters (initialDelaySeconds, periodSeconds, timeoutSeconds, failureThreshold) are appropriately configured for each application's startup and response characteristics.  
* **Zero-Downtime Deployment:** Rolling updates of the application should complete without any observable dropped requests or errors from the client's perspective.

## **Implementation Details**

### **1. Application Code Changes**

a. SIGTERM Handler:  
Implement a signal handler for SIGTERM (signal 15).

* **Action on SIGTERM:**  
  * Set an internal flag (e.g., isShuttingDown = true).  
  * Stop listening for new incoming connections/requests (e.g., close the server's listening socket).  
  * Start an **application-defined shutdown timer (60 seconds)**.  
  * Begin graceful draining: allow all currently active requests to complete.  
  * Once all active requests are finished (or the 60-second timer expires), perform any necessary cleanup (e.g., close database connections, flush logs) and then exit the process with code 0.

b. /readyz Endpoint (Readiness Probe):  
This endpoint should reflect the application's ability to serve new requests.

* **Logic:**  
  * Check the isShuttingDown flag: If true, immediately return 503 Service Unavailable.  
  * **Perform lightweight checks for *all* critical external dependencies (e.g., database, Redis, message queue, essential external APIs). These checks should be executed in parallel (e.g., using goroutines, async/await, or thread pools) for efficiency.**  
  * **Implement Tolerance:** For each dependency, maintain a counter for consecutive failures. Only mark the dependency as "unready" if it fails N consecutive times (e.g., 2-3 times). This prevents transient network hiccups from immediately taking the pod out of service.  
  * If all checks pass, return 200 OK. Otherwise, return 503 Service Unavailable.

c. /healthz Endpoint (Liveness Probe):  
This endpoint should be a very quick check to ensure the application process is alive and not deadlocked.

* **Logic:**  
  * Perform a minimal check, e.g., return 200 OK if the main application thread is responsive.  
  * **Do NOT** check external dependencies here, as their temporary unavailability should not cause a restart of the application process itself.  
  * This endpoint should continue to return 200 OK even if the application is in the process of gracefully shutting down (after SIGTERM), as long as it's still draining requests.

### **2. Kubernetes Configuration (Deployment YAML)**

Update your Deployment manifest (e.g., my-api-deployment.yaml) as follows:

```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: my-api-deployment  
  labels:  
    app: my-api  
spec:  
  replicas: 3 # Adjust as needed  
  selector:  
    matchLabels:  
      app: my-api  
  template:  
    metadata:  
      labels:  
        app: my-api  
    spec:  
      # Optional: Use an init system like dumb-init to ensure SIGTERM is properly propagated  
      # to your application process, especially if your Dockerfile uses shell scripts.  
      # entrypoint: ["/usr/bin/dumb-init", "--"]  
      # command: ["/app/your-api-executable"]  
      containers:  
      - name: my-api-container  
        image: your-registry/your-api-image:latest # Replace with your actual image  
        ports:  
        - containerPort: 8080 # Adjust to your application's port  
          
        # --- Liveness Probe Configuration ---  
        livenessProbe:  
          httpGet:  
            path: /healthz # Path to your liveness endpoint  
            port: 8080     # Port your application listens on  
          initialDelaySeconds: 30 # Time to wait before first liveness check (allow startup)  
          periodSeconds: 10     # How often to perform the check (every 10 seconds)  
          timeoutSeconds: 5     # How long to wait for a response before timing out  
          failureThreshold: 3   # Number of consecutive failures before restarting the container  
          
        # --- Readiness Probe Configuration ---  
        readinessProbe:  
          httpGet:  
            path: /readyz  # Path to your readiness endpoint  
            port: 8080     # Port your application listens on  
          initialDelaySeconds: 5  # Time to wait before first readiness check (can be shorter than liveness)  
          periodSeconds: 5      # How often to perform the check (every 5 seconds)  
          timeoutSeconds: 3     # How long to wait for a response before timing out  
          failureThreshold: 1   # Number of consecutive failures before marking pod as unready  
                                # Set to 1 if your /readyz handles internal tolerance, otherwise higher.  
          
        # --- Graceful Termination Period ---  
        # This is the maximum time Kubernetes will wait after sending SIGTERM before sending SIGKILL.  
        # It should be > application's shutdown timeout (60s) to give it a safety margin.  
        terminationGracePeriodSeconds: 75 # Kubernetes will wait 75 seconds.  
                                          # Your application should exit within 60 seconds.
```

**Explanation of Parameters:**

* initialDelaySeconds: How long Kubernetes waits after the container starts before initiating the first probe. This is crucial for applications that take time to start.  
* periodSeconds: How frequently Kubernetes performs the probe.  
* timeoutSeconds: How long Kubernetes waits for a response from the probe endpoint. If no response is received within this time, the probe is considered a failure.  
* failureThreshold: The number of consecutive probe failures required for Kubernetes to take action (restart for liveness, remove from service for readiness).  
* terminationGracePeriodSeconds: The maximum time Kubernetes will wait for your application to shut down gracefully after sending SIGTERM. If the process is still running after this period, Kubernetes sends SIGKILL. **This is set to 75 seconds, providing a 15-second buffer beyond your application's intended 60-second shutdown timeout.**

## **Testing Strategy**

1. **Local Testing:**  
   * Run the application locally.  
   * Send SIGTERM to the application process.  
   * Verify that `/readyz` immediately returns non-2xx.  
   * Verify that `/healthz` continues to return 200 OK for the duration of the application's graceful shutdown.  
   * Send test requests to the application: new requests should be rejected, existing requests should complete within the 60-second timeout.  
   * Verify the application exits cleanly after draining within 60 seconds.  
2. **Kubernetes Deployment Testing:**  
   * Deploy the updated application to a staging environment.  
   * Monitor application logs for graceful shutdown messages.  
   * **Rolling Update Test:** Perform a `kubectl rollout restart deployment/my-api-deployment`.  
     * Continuously send requests to the service IP and monitor for any dropped requests or 5xx errors from the client side.  
     * Use `kubectl get pods -w` to observe pod transitions (Running -> Terminating -> Running).  
     * Use `kubectl describe pod <old-pod-name>` to see probe events and SIGTERM receipt, verifying that the pod enters Terminating state and its Ready condition becomes False while Liveness remains True during the drain period.  
   * **Scale Down Test:** Scale down the deployment (e.g., `kubectl scale deployment/my-api-deployment --replicas=1`).  
     * Observe the graceful termination of scaled-down pods.  
   * **Unhealthy Dependency Simulation:** Temporarily make a critical dependency (e.g., database) unavailable.  
     * Verify that `/readyz` returns non-2xx for the affected pods after the internal tolerance threshold is met.  
     * Verify that traffic is no longer routed to these pods.  
     * Verify that `/healthz` remains 200 OK.  
     * Restore the dependency and verify pods become ready and receive traffic again.

## **Considerations**

* **Logging:** Ensure comprehensive logging within your application for SIGTERM receipt, request draining progress, and dependency health checks.  
* **dumb-init / tini:** If your Docker images use shell scripts as the entrypoint, consider using dumb-init or tini to ensure SIGTERM is correctly propagated to your main application process.  
* **Probe Endpoint Simplicity:** Keep health check endpoints as lightweight and fast as possible to avoid adding overhead or causing false failures.  
* **terminationGracePeriodSeconds Value:** This value is a hard limit. Your application *must* exit within this time. The 75-second setting provides a safety net for your 60-second application shutdown.

This ticket provides a clear and detailed plan for implementing these critical features.
