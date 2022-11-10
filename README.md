## Project setup
1. Setup docker on machine
  * `apt update && apt full-upgrade -y`
  * `apt install docker.io docker-compose -y`
1. Put the php code into /app folder and run
1. Run: `docker compose up -d`
1. Logs: `docker compose logs -f`

## Project locale dev setup
1. `docker compose -f docker-compose.dev.yaml up -d`
1. Enter the /app folder and adjust live your code