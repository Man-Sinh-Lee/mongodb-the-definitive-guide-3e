### Chapter 19: An Introduction to MongoDB Security

Security is a critical aspect of any database system, and MongoDB provides robust mechanisms for both **authentication** and **authorization**, ensuring that only authorized users and systems can access data and perform operations. This chapter covers MongoDB’s security model, including authentication methods, authorization strategies, and transport layer encryption using x.509 certificates.

#### MongoDB Authentication and Authorization

##### Authentication Mechanisms

MongoDB supports various authentication mechanisms to verify the identity of users and systems accessing the database. The main authentication mechanisms include:

- **SCRAM (Salted Challenge Response Authentication Mechanism)**: The default authentication method used by MongoDB, which hashes passwords and secures user credentials. SCRAM is available in two variants, `SCRAM-SHA-1` and `SCRAM-SHA-256`, with `SCRAM-SHA-256` being the stronger and recommended option for most use cases.

  Example of enabling SCRAM authentication:
  
  ```js
  mongod --auth --setParameter authenticationMechanisms=SCRAM-SHA-256
  ```

- **x.509 Certificates**: Used to authenticate both MongoDB clients and members of a replica set or sharded cluster. This method uses certificates issued by a trusted certificate authority (CA).

- **LDAP (Lightweight Directory Access Protocol)**: MongoDB supports integration with LDAP for external user authentication, allowing centralized user management.

- **Kerberos**: Another supported authentication method, typically used in environments that require single sign-on (SSO) capabilities.

##### Authorization

Once authenticated, **authorization** ensures that users only have access to the actions and data they are permitted to interact with. MongoDB implements a **role-based access control (RBAC)** system where roles define what operations users can perform. Common roles include:

- **read**: Allows users to read data in a specific database.
- **readWrite**: Allows users to read and write data in a specific database.
- **dbAdmin**: Provides administrative capabilities, such as index creation and database stats viewing.
- **clusterAdmin**: Grants permissions for managing an entire MongoDB cluster, including replica sets and sharding configurations.

For example, to create a user with `readWrite` access to a database:

```js
db.createUser({
  user: "dbUser",
  pwd: "password123",
  roles: [ { role: "readWrite", db: "myDatabase" } ]
});
```

##### Using x.509 Certificates to Authenticate Both Members and Clients

MongoDB allows x.509 certificate-based authentication for both clients and members of replica sets or sharded clusters. x.509 authentication relies on certificates issued by a trusted CA. The process ensures that only authorized users and machines with valid certificates can access the MongoDB instance.

To enable x.509 authentication, MongoDB must be started with the `--auth` and `--tlsMode` options. You also need to configure the use of CA-signed certificates for both clients and members of replica sets.

Example command for starting MongoDB with x.509 authentication:

```js
mongod --tlsMode requireTLS --tlsCertificateKeyFile /path/to/server.pem --tlsCAFile /path/to/ca.pem --auth
```

#### A Tutorial on MongoDB Authentication and Transport Layer Encryption

Transport layer encryption ensures that all communication between clients, servers, and replica set members is encrypted using TLS/SSL. This section outlines a step-by-step guide to set up MongoDB authentication and encryption using x.509 certificates.

##### Establish a CA (Certificate Authority)

The first step is to establish a trusted CA. If you don’t have a trusted external CA, you can generate your own internal CA for testing purposes using OpenSSL.

```js
openssl genpkey -algorithm RSA -out ca-key.pem
openssl req -new -x509 -key ca-key.pem -out ca-cert.pem -days 365
```

This creates a self-signed CA certificate that can be used to sign member and client certificates.

##### Generate and Sign Member Certificates

Next, generate x.509 certificates for each MongoDB server that will be part of the replica set or cluster.

```js
openssl req -newkey rsa:2048 -nodes -keyout server-key.pem -out server-req.pem
openssl x509 -req -in server-req.pem -CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 -out server-cert.pem -days 365
```

This generates and signs the server’s certificate using the CA’s certificate and key.

##### Generate and Sign Client Certificates

Similarly, client certificates must be generated and signed. Clients authenticate using x.509 certificates just like the MongoDB members.

```js
openssl req -newkey rsa:2048 -nodes -keyout client-key.pem -out client-req.pem
openssl x509 -req -in client-req.pem -CA ca-cert.pem -CAkey ca-key.pem -set_serial 02 -out client-cert.pem -days 365
```

The client will use this certificate to securely connect to MongoDB instances.

##### Bring Up the Replica Set Without Authentication and Authorization Enabled

Before enabling full authentication and encryption, it’s a good practice to set up the replica set without these security features to ensure everything is functioning correctly. Start MongoDB instances as a basic replica set without the `--auth` or `--tlsMode` options:

```js
mongod --replSet "rs0" --port 27017 --dbpath /data/db1
```

Initiate the replica set in the MongoDB shell:

```js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019" }
  ]
});
```

##### Create the Admin User

Once the replica set is functioning properly, create the admin user. This user will have the necessary privileges to manage the MongoDB instance and enforce authentication and authorization.

```js
use admin
db.createUser({
  user: "admin",
  pwd: "admin123",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "root" ]
});
```

This user will have the `root` role, which provides full administrative access across the MongoDB instance.

##### Restart the Replica Set with Authentication and Authorization Enabled

Now that the admin user is created, restart each MongoDB instance with authentication and TLS encryption enabled. This is done by adding the `--auth` and `--tlsMode requireTLS` options.

```js
mongod --replSet "rs0" --port 27017 --dbpath /data/db1 --auth --tlsMode requireTLS --tlsCertificateKeyFile /path/to/server.pem --tlsCAFile /path/to/ca.pem
```

Do this for all members of the replica set. Clients connecting to the MongoDB instance must now provide their x.509 certificates for authentication:

```js
mongo --tls --tlsCertificateKeyFile /path/to/client.pem --tlsCAFile /path/to/ca.pem --host "localhost" --port 27017
```

With both authentication and encryption enabled, MongoDB ensures that only authorized users and machines can access the database securely.

---

This chapter highlights MongoDB’s comprehensive security features, focusing on authentication, authorization, and transport layer encryption. By implementing security best practices—such as enabling role-based access control (RBAC), using x.509 certificates for authentication, and securing data transmission with TLS—MongoDB users can protect their database from unauthorized access and ensure data integrity across the network.