

If you are dealing with MYSQL with a large number of read queries such as a shopping site, you can make the performance more powerful by setting up read replicas 
of your master server, which is a lifesaver.


Whenever a change occurs on the Master server, it will be reflected in the replicas too.

Using Amazon RDS, you can easily set up a master & a read replica model.

Whenever the master fails to work, one of the replicas will be promoted to master for a seamless service.

There are a few advantages of using RDS server over setting up an instance with MySQL manual installation.

In RDS, you don't need to install or worry about the MySQL server, it will be taken care of by amazon.

The headache is low, you can concentrate on your application.

Let's make a simple MySQL read replica model and load balance it with route 53.

Our setup will be like the following.

![Untitled Diagram drawio (3)](https://user-images.githubusercontent.com/61390678/215254401-8ec74c44-4c32-477f-911c-551e7b045c9d.png)

FIrst of all we need to create master server for our application.This can be done from the RDS panel inside the AWS control panel.
You will get different type of MySQL servers here, Since this is a sample setup we are proceeding with MySQL server with version 5.7.

So Let's start setting up this.

1.Creating the master server
=========================
First we need to create a mysql master server. For that login to your RDS console.

![image](https://user-images.githubusercontent.com/61390678/215254457-1204edfa-8b78-433e-839e-725e8baec351.png)


Click on create database and choose engine type. As mentioned before we are setting up this with mysql .You can choose the correct version which fits to you .

Next you need to choose the environment, Since we are testing the setup we can use either free tier or test/dev environment.

In the next step you need to create the Master login details. The default user is admin and for the testing purpose we are setting up the password as admin12345.

Choose the instance type as per your requirement. You can leave the rest of the settings, That will be fine.


It's better to enable backup option for your RDS instance. You can configure the backup retension in the additional settings ,then click on create.

![image](https://user-images.githubusercontent.com/61390678/215254499-8f135ffb-bce9-4191-b74c-bb494a978509.png)

Once it is done you can connect to your instance via the endpoint .

![image](https://user-images.githubusercontent.com/61390678/215254533-63275d4b-d5e5-459e-a22a-3412bc53d899.png)

![image](https://user-images.githubusercontent.com/61390678/215254582-c7145363-d6e8-41cb-9da1-07af94c2ad17.png)

Add a few entries in the db server

```
MySQL [(none)]> create database users;
Query OK, 1 row affected (0.00 sec)

MySQL [(none)]>
MySQL [(none)]> use users;
Database changed

MySQL [users]> CREATE TABLE details (id int(5), name varchar(20), age int(3) );
Query OK, 0 rows affected (0.02 sec)

MySQL [users]>
MySQL [users]> INSERT INTO details (id,name,age) VALUES(1,"user1",20);
Query OK, 1 row affected (0.01 sec)

MySQL [users]> INSERT INTO details (id,name,age) VALUES(2,"user2",22);
Query OK, 1 row affected (0.00 sec)

MySQL [users]> INSERT INTO details (id,name,age) VALUES(3,"user3",23);
Query OK, 1 row affected (0.00 sec)

MySQL [users]> INSERT INTO details (id,name,age) VALUES(4,"user4",24);
Query OK, 1 row affected (0.01 sec)

MySQL [users]>
MySQL [users]> select * from details;
+------+-------+------+
| id   | name  | age  |
+------+-------+------+
|    1 | user1 |   20 |
|    2 | user2 |   22 |
|    3 | user3 |   23 |
|    4 | user4 |   24 |
+------+-------+------+
4 rows in set (0.00 sec)

```

Once it is done, you can create the read replicas of your master server.

2.Creating read replica
======================
In order to do this, click on your db, on the right click on actions, choose create read replica.select the source db and add a name to your database replica.

Complete the setup and click on create.

![rp1](https://user-images.githubusercontent.com/61390678/215254706-fe14c3f7-d5f3-4c1d-927c-3798118fab9c.png)


You can repeat the process as per your required number of replicas.here we are setting up 2 replicas.

![all reps](https://user-images.githubusercontent.com/61390678/215254723-df6efdbf-82be-41c5-a711-9085eed99acd.png)


Our next task is to loadbalance the read replicas. We can use route 53 for that. We aew using weighted policy here.

3.Setting up load-balancing with route 53
=========================================

In order to set up loadbalancing with route 53, we are setting up a subdomain called replica in our main domain.

Login to your aws route 53 console, choose the zone and click on add record.

set the record name as replica,and choose the record type as CNAME, in the value section you need to enter the end point value of the read-replica 1.

Change the TTL to  0, since we don't need to cache the values.

In the routing policy ,we need to setup a routing based on weight,since we are using 2 copies of our read replica, we are setting it as 50.Then add a name to the ID.
![image](https://user-images.githubusercontent.com/61390678/215254855-77260a22-8377-4ab1-8034-02640fdaee7c.png)

Repeat the same process for the second replica, only need to change the end point value.Kindly wait for few minutes for the DNS propagation.

Once you added the records, you can see them on under the records area.

![image](https://user-images.githubusercontent.com/61390678/215254883-6ef36d5b-f385-483f-9b76-48b14be08afe.png)

Now lets check the loadbalancing results.

![image](https://user-images.githubusercontent.com/61390678/215254931-e9b85f28-3e72-4d49-9423-24b7f05372fa.png)


As you could see that we are getting the results from both the servers and it shows that our setup  is up and running.

Thanks for reading :)
