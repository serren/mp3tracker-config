Git-backed configuration repository for the mp3tracker Config Server.                                                                                                     
                                                                                                                                                                            
  Spring Cloud Config Server reads YAML files from this repo and serves them to client services at startup and on demand via `POST /actuator/refresh`.                      
                                                     
  ## Files                               

  | File | Applies to |
  |---|---|
  | `application.yml` | All client services (shared defaults) |
  | `resource-service.yml` | `resource-service` only |
  | `song-service.yml` | `song-service` only |

  Per-service files are merged on top of `application.yml` — service-specific values take precedence.

  ## Property source priority

  OS environment variables  (highest)
    └── Config Server (this repo)
          └── Service own application.yml  (lowest)

  Secrets (DB passwords, S3 keys, RabbitMQ credentials) are never stored here — they are injected via env vars in `compose.yaml`.

  ## Contents

  ### `application.yml` — shared across all clients

  - Eureka heartbeat / fetch intervals
  - Actuator endpoints (`health`, `refresh` exposed)

  ### `resource-service.yml`

  - Eureka service URL
  - RabbitMQ exchange, queue, and routing key names
  - Retry policy (max attempts, backoff intervals)
  - Log level for `com.example`

  ### `song-service.yml`

  - Eureka service URL
  - Log level for `com.example`

  ## Dynamic refresh

  To change a property at runtime without restarting:

  ```bash
  # 1. Edit the relevant YAML file, e.g.:
  #    logging.level.com.example: DEBUG  (in resource-service.yml)

  # 2. Commit the change
  git commit -am "update config"

  # 3. Trigger refresh on the target service
  curl -X POST http://localhost:8081/actuator/refresh   # resource-service
  curl -X POST http://localhost:8082/actuator/refresh   # song-service

  # Response: list of changed property keys
  # ["logging.level.com.example"]

  Only @RefreshScope beans and externalized properties (e.g. logging.level.*) are re-applied without a restart.
