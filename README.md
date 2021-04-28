# Vault Transit Datakey Example

## Setup

You need a database to test with:

```sh
docker pull mysql/mysql-server:5.7
mkdir ~/transit-data
docker run --name mysql-transit \
  -p 3306:3306 \
  -v ~/transit-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_ROOT_HOST=% \
  -e MYSQL_DATABASE=my_app \
  -e MYSQL_USER=vault \
  -e MYSQL_PASSWORD=vaultpw \
  -d mysql/mysql-server:5.7
```

To configure Vault:

```sh
vault server -dev -dev-root-token-id=root
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="root"
vault login root
vault write sys/license text="$VAULT_LICENSE"
vault secrets enable transit
vault write -f transit/keys/my_app_key
```

## DEMO

- run the app:

  ```sh
  VAULT_TOKEN=root go run main.go
  ```

- view App UI http://localhost:1234
- create a few records, use pictures or will error
- inspect the contents of the database in UI
- use CLI to inspect database records

```sh
docker exec -it mysql-transit mysql -uroot -proot

# inside mysql
USE my_app;
SELECT * FROM user_data LIMIT 10;
# look at the data returned and observe the encryption

SELECT user_id, file_id, mime_type, file_name FROM user_files LIMIT 10;
# if you select the file contents itself say good bye to your terminal...
```
