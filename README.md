# Mongodb_cluster(do this setup on root account)

# Step 1: Install docker on Amazon Linux
cmd - yum install docker

# Step 2: Start docker service
cmd - systemctl start docker

# Step 3: using docker pull mongodb service
cmd - docker pull mongo

# Step 4 : installing mongodb services on amazon linux
cmd - vim /etc/yum.repos.d/mongodb-org-7.0.repo
# code : (shift + : + wq to exit)
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2023/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://pgp.mongodb.com/server-7.0.asc

# Step 5 : installing mongodb client set command
cmd - yum install mongodb-org

# Step 6 : Installing docker-compose to lunch multiple containers
1st cmd - sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d'"' -f4)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
2nd cmd - sudo chmod +x /usr/local/bin/docker-compose

# Step 7 : Setting up config sever 
cmd - vim config.yml (Put code for config.yml)
cmd - docker-compose -f config.yml up -d

# Step 8 : Performing election to select primary config server (connect to any of the config server container using mongosh mongodb://AmazonlinuxPrivateTP:40001)
mongodb cmd - 
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "172.31.0.6:40001"}
    ]
  }
)

rs.status()

# Step 9 : Setting up Shrad 1 
cmd - vim shrad1.yml (Put code for shrad1.yml)
cmd - docker-compose -f shrad1.yml up -d

# Step 10 : Performing election to select primary shrad 1 (connect to any of the shrad 1 container using mongosh mongodb://AmazonlinuxPrivateTP:50001)
mongodb cmd - 
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "172.31.0.6:50001" }
    ]
  }
)

rs.status()

# Step 11 : Setting up Shrad 2
cmd - vim shrad2.yml (Put code for shrad2.yml)
cmd - docker-compose -f shrad2.yml up -d

# Step 12 : Performing election to select primary shrad 2 (connect to any of the shrad 2 container using mongosh mongodb://AmazonlinuxPrivateTP:50001)
mongodb cmd - 
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "172.31.0.6:50004" }
    ]
  }
)

rs.status()

# Step 13 : Creating router server and connecting to config server
cmd - vim router.yml (Put code for router.yml)

# Step 14 : Enable sharding in router server
1. connect to router mongosh and use this mongosh cmd -
-> sh.addShard("shard1rs/172.31.0.6:50001")
-> sh.addShard("shard2rs/172.31.0.6:50004")
-> sh.status()
2. Enabling shrading for collections :
-> sh.enableSharding("DATABASENAME")
-> sh.shradCollection("DATABASENAME.COLLECTION_NAME",{"_id":1})
