### Chapter 21: Setting Up MongoDB in Production

When deploying MongoDB in production, it is essential to follow best practices for starting, stopping, securing, and monitoring your MongoDB instances. This chapter outlines how to properly configure and manage MongoDB for a production environment.

#### Starting from the Command Line

Starting MongoDB in production involves configuring the `mongod` process with appropriate options to ensure that the database is secure, performant, and properly connected to the network.

Basic example of starting MongoDB from the command line:

```js
mongod --config /etc/mongod.conf
```

MongoDB is typically started using a configuration file (`mongod.conf`), which allows specifying multiple options like data directories, network interfaces, logging, and security settings in a structured format.

Common configuration options include:

- **`--dbpath`**: Specifies the directory where MongoDB stores data files. In production, ensure this is on a reliable and high-performance disk:

  ```js
  mongod --dbpath /var/lib/mongodb
  ```

- **`--logpath`**: Specifies where MongoDB stores log files. Production deployments typically direct logs to a dedicated file:

  ```js
  mongod --logpath /var/log/mongodb/mongod.log --logappend
  ```

- **`--fork`**: In production, MongoDB is typically started as a background process, achieved by using the `--fork` option:

  ```js
  mongod --config /etc/mongod.conf --fork
  ```

- **`--bind_ip`**: By default, MongoDB binds to `localhost`, but in production environments, you'll need to specify which IP addresses the database should listen to:

  ```js
  mongod --bind_ip 192.168.1.100
  ```

For added security, you should always specify the appropriate network interfaces and disable the default `localhost` binding.

#### Stopping MongoDB

Stopping MongoDB gracefully ensures that all data is properly written to disk, and the system is left in a consistent state.

MongoDB can be stopped safely using the following methods:

- **`SIGTERM` Signal**: You can stop MongoDB by sending the **`SIGTERM`** signal to the `mongod` process, which will safely shut down the server:

  ```js
  sudo kill -SIGTERM <mongod_pid>
  ```

- **Using the `mongo` Shell**: You can connect to the `mongo` shell and use the `shutdown` command to stop the MongoDB instance gracefully:

  ```js
  use admin
  db.shutdownServer()
  ```

- **Systemd or Init Scripts**: On Linux systems using **systemd** or **init.d**, you can use the service management commands:

  ```js
  sudo systemctl stop mongod
  ```

  or

  ```js
  sudo service mongod stop
  ```

#### Security

Security is a critical aspect of deploying MongoDB in production. Proper encryption, secure network communication, and role-based access control (RBAC) must be configured to protect sensitive data.

##### Data Encryption

In production environments, it is highly recommended to enable **data encryption** to protect sensitive information at rest. MongoDB supports **encryption at rest** through its **WiredTiger** storage engine. Enabling encryption ensures that all data stored on disk is encrypted using a specified key.

To enable encryption, add the following options to the `mongod.conf` file:

```yaml
security:
  enableEncryption: true
  encryptionKeyFile: /path/to/encryption-keyfile
```

This configuration enables encryption and uses the specified keyfile to encrypt and decrypt data. You can generate a keyfile using the OpenSSL command:

```js
openssl rand -base64 32 > /path/to/encryption-keyfile
```

Make sure to protect the keyfile with the correct permissions so that only the MongoDB process can access it.

##### SSL Connections

To secure data in transit, MongoDB supports **SSL/TLS** encryption, which ensures that all communications between clients and servers, as well as between members of replica sets or sharded clusters, are encrypted.

To enable SSL in MongoDB, the following options are used:

```yaml
net:
  ssl:
    mode: requireSSL
    PEMKeyFile: /etc/ssl/mongodb.pem
    CAFile: /etc/ssl/ca.pem
```

This configuration ensures that MongoDB requires SSL for all connections and uses the specified certificate and CA files to authenticate clients and servers.

Example of starting `mongod` with SSL from the command line:

```js
mongod --sslMode requireSSL --sslPEMKeyFile /etc/ssl/mongodb.pem --sslCAFile /etc/ssl/ca.pem
```

For client connections, SSL can be enforced as follows:

```js
mongo --ssl --host "myMongoServer" --sslCAFile /etc/ssl/ca.pem --sslPEMKeyFile /etc/ssl/client.pem
```

Using SSL ensures that all data transmitted between MongoDB instances and clients is encrypted, protecting it from interception.

#### Logging

MongoDB provides extensive logging capabilities, which are essential for monitoring and troubleshooting in production environments.

##### Configuring Log Output

Logs in MongoDB provide detailed information about operations, connections, and system performance. By default, MongoDB logs are stored in `/var/log/mongodb/mongod.log`, but you can configure a different path using the `logpath` option in the `mongod.conf` file:

```yaml
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true  # Continues to append to the log file instead of overwriting it
  verbosity: 1     # Increases verbosity for more detailed logs (0 is the default level)
```

Setting `logAppend: true` ensures that MongoDB continues logging in the same file instead of overwriting logs when the process is restarted. You can adjust the **verbosity** to include more detailed information in the logs, which can be helpful for troubleshooting.

##### Analyzing Logs

MongoDB logs can be used to monitor system health, detect performance issues, and diagnose errors. Example log entries may include connection errors, slow queries, or replica set election events.

To view real-time logs, use the `tail` command:

```js
tail -f /var/log/mongodb/mongod.log
```

For more granular control over log verbosity, you can adjust the logging level dynamically using the MongoDB shell:

```js
db.adminCommand({ setParameter: 1, logLevel: 2 })
```

This increases the log level to provide more detailed information, which is useful for diagnosing issues during production.

#### Monitoring Log Rotation

Log files can grow large over time, especially in busy production environments. MongoDB allows you to rotate logs manually or configure automatic log rotation using the **logrotate** tool. To manually rotate logs, you can use the following command:

```js
db.adminCommand({ logRotate: 1 })
```

This closes the current log file and opens a new one, which is useful for managing log size and keeping logs organized.

---

In summary, deploying MongoDB in production requires careful configuration of the `mongod` process, proper shutdown procedures, and a strong focus on security through data encryption and SSL connections. Logging is crucial for monitoring and maintaining the health of the database, and MongoDB provides flexible logging options for troubleshooting and auditing purposes. By following best practices for starting, stopping, securing, and logging in MongoDB, administrators can ensure a robust and secure production environment.