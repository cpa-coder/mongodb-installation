# MongoDB Installation Setup for Windows

Configure MongoDB server with authentication and replica set.

:warning:***Caution:*** This will prevent you from running MongoDB server as a window service.

## Table of Contents

- [MongoDB Installation Setup for Windows](#mongodb-installation-setup-for-windows)
  - [Table of Contents](#table-of-contents)
  - [Software Application Requirements](#software-application-requirements)
  - [Step-by-step Instruction](#step-by-step-instruction)
  - [Database Backup and Restore](#database-backup-and-restore)
  - [Disclaimer](#disclaimer)

## Software Application Requirements

1. [OpenSSL v3](http://slproweb.com/download/Win64OpenSSL_Light-3_0_0.exe)
2. [MongoDB Server Community Edition](https://www.mongodb.com/try/download/community)
3. [Mongo Shell](https://downloads.mongodb.com/compass/mongosh-1.1.6-x64.msi)
4. [MongoDb Database Tools](https://www.mongodb.com/try/download/database-tools)

## Step-by-step Instruction

1. Install all the [required applications](#application-requirements). Make sure to check if environment variables are properly setup/installed.

2. Create key file "`C:\mongodb\keyfile\key.pem`". Run the following commands:

    ```ps
    mkdir C:\mongodb\keyfile
    
    openssl rand -base64 756 > C:\mongodb\keyfile\key.pem

    icacls.exe C:\mongodb\keyfile\key.pem /reset
    icacls.exe C:\mongodb\keyfile\key.pem /GRANT:R "$($env:USERNAME):(R)"
    icacls.exe C:\mongodb\keyfile\key.pem /inheritance:r
    ```

    ***Note:*** You may change filename and file path location as much as you like.

3. Make custom folders where to save MongoDB data and log file. You can manually create those or use the following commands:

    ```ps
    mkdir C:\mongodb\data
    mkdir C:\mongodb\log
    ```

4. Open `mongod.cfg` in "`C:\Program Files\MongoDB\Server\<version>\bin`" using text editor with admin previledge and update the following properties:

    ```ps
    storage
      dbPath: C:\mongodb\data

    systemLog
      path: C:\mongodb\log\mongod.log

    net
      bindIp: 0.0.0.0
   
    ```

5. Restart [MongoDB](https://www.mongodb.com/try/download/community) server from Services, if installed as window service, or run the following command to start:

    ```ps
    mongod --config "C:\Program Files\MongoDB\Server\<version>\bin\mongod.cfg"
    ```

    ***Note:*** Make sure `security` property in `mongod.cfg` is not set or disabled first.

6. Run `mongo shell` to add new user with root previlege with the following command:

    ```ps
    mongosh admin --eval "db.createUser({user:'user', pwd:'password', roles:[{role:'root', db:'admin'}]})"
    ```

    ***Note:*** Change user and password as needed.

    To check newly added user, run the following commands:

    ```ps
    mongosh
    use admin
    db.getUsers()
    ```

7. Now its time to enable authentication and replica set. Again, open `mongod.cfg` using text editor with admin previledge and update the following properties:

    ```ps
    security:
      authentication: enabled
      keyFile: C:\mongodb\keyfile\key.pem

    replication:
      replSetName: "<name>"
    ```

    ***Note:*** You may set any name for `replSetName` property.

8. If you can run MongoDB as window service or planning to run MongoDB as a window service, you skip this step.

   To uninstall MongoDB service, run the following commands:

    ```ps
    #If service is running
    net stop MongoDB

    sc.exe delete MongoDB
    ```

9. Check server configuration setup by running the following command:

    ```ps
    mongod --config "C:\Program Files\MongoDB\Server\<version>\bin\mongod.cfg"
    ```

10. Once `mongod` is running from the terminal, open another terminal and run `mongo shell` with the following commands:

    ```ps
    #Supply user and password setup on step 6.
    mongosh -u user -p password

    #Once mongo shell is up and running, intiate replica set (this can only be done once)
    rs.initiate()
    ```

11. Congratulations!!! You have successfully configure MongoDB Server :tada::clap::clap::clap::clap:

---

## Database Backup and Restore

Make sure you install [MongoDb Database Tools](https://www.mongodb.com/try/download/database-tools)

- Backup MongoDB database

    To back MongoDB database use the following command:

    ```ps
    #To create directory
    mkdir "C:\back-up"
    cd "C:\back-up"

    mongodump -u <user> -p <password> --authenticationDatabase admin --db <database name> 
    ```

    You can also specified output location with "`--out=<output folder path>`".

    ***Note:*** When output directory is not specified backup database will be created in "`.\dump\<database name>`" folder.

- Restore MongoDB database

    To restore MongoDB database use the following command

    ```ps
    mongorestore -u <user> -p <password> --authenticationDatabase admin --db <database name> <database folder path>
    ```

## Disclaimer

This is created for personal use. In no event shall the author be liable for any claim, damages or other liability in connection with this instruction. ***Use this at your OWN RISK.***:warning:
