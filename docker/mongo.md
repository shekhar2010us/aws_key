## Create a directory to persist data

```
mkdir -p /home/ubuntu/data
```


## Start a mongo `server` instance

```
docker run -d -p 27017:27017 -v ~/data:/data/db mongo
```

```
# Install the MongoDB client
sudo apt-get install mongodb-clients
```

## Connect to MongoDB from local
<p>There are multiple ways to connect. </p>

```
private_ip:private_port
public_ip:public_port
```

you can get private_ip of the container by inspecting the container.

```
# connect
mongo localhost/mydb
mongo <aws_ip>:27017/mydb
mongo <container_private_ip>:27017/mydb

# insert to mongo
db.createCollection('cities')
db.cities.insert({ name: 'New York', country: 'USA' })
db.cities.insert({ name: 'Paris', country: 'France' })
db.cities.find()

## check /home/ubuntu/data folder
```

