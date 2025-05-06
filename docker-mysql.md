
```sh
docker exec -it db mysqldump --skip-add-drop-table -h test.com -u -p'xxx' database > a.dump
cat a.dump | docker exec -it db mysql -u user -p'x' database
```
