### Liveness probe and readiness probe
- How kubernetes know the application has issue and requires restart?
- Needs a healthcheck for self heal.

***Liveness:*** Continiously checks if application is ready.
***Readyness:*** Checks if service ready to serve traffic.
                 Application tempororily unable to serve traffic.
                 May need to load large data or config file during startup.
                 Or depen on external services after startup.
                 Readiness probes detect this, will not kill the app.
                 The pods will no receive traffic from services.


