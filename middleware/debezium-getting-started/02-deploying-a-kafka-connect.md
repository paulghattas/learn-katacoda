We are going to use [binary build](https://docs.openshift.org/latest/dev_guide/dev_tutorials/binary_builds.html) feature of OpenShift together with [distribution](http://central.maven.org/maven2/io/debezium/debezium-connector-mysql/0.7.4/) of MySQL [plugin](http://debezium.io/docs/connectors/mysql/) to deploy a Kafka Connect node with Debezium MySQL plugin embedded.

![Debezium deployment](../../assets/middleware/debezium-getting-started/deployment-step-2.png)

**1. Deploy an empty Kafka Connect node**

We will again use a [template](https://raw.githubusercontent.com/strimzi/strimzi/master/examples/templates/cluster-controller/connect-s2i-template.yaml) from Strimzi project that is already installed in the environment.
As with Kafka broker we need to configure the template to support single-instance-only deployment.

``oc new-app strimzi-connect-s2i -p KAFKA_CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR=1 -p KAFKA_CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR=1 -p KAFKA_CONNECT_STATUS_STORAGE_REPLICATION_FACTOR=1``{{execute}}

A new build config will be created

``oc get bc``{{execute}}

    NAME                         TYPE      FROM      LATEST
    my-connect-cluster-connect   Source    Binary    1

and a new service is deployed.

``oc get svc -l app=strimzi-connect-s2i``{{execute}}

    NAME                         CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    my-connect-cluster-connect   172.30.29.20   <none>        8083/TCP   12m

**2. Embed Debezium into Connect**

We will use a binary build to create a Connect node with a Debezium plugin inside

``oc start-build my-connect-cluster-connect --from-archive http://central.maven.org/maven2/io/debezium/debezium-connector-mysql/0.7.4/debezium-connector-mysql-0.7.4-plugin.tar.gz --follow``{{execute}}

The Connect node should be redeployed after the build completes

``oc get pods -w -l app=strimzi-connect-s2i``{{execute}}

    NAME                                 READY     STATUS        RESTARTS   AGE
    my-connect-cluster-connect-1-wktnt   0/1       Terminating   0          50s
    my-connect-cluster-connect-2-lqqht   0/1       Running       0          26s

**3. Verify that Connect is up and contains Debezium**

Wait till the Connect node is ready

    NAME                                 READY     STATUS    RESTARTS   AGE
    my-connect-cluster-connect-2-lpnd2   1/1       Running   0          1m

and list all plug-ins available for use.

``oc exec my-cluster-kafka-0 -- curl -s http://my-connect-cluster-connect:8083/connector-plugins``{{execute}}

    [
        {"class":"io.debezium.connector.mysql.MySqlConnector","type":"source","version":"0.7.4"},
        {"class":"org.apache.kafka.connect.file.FileStreamSinkConnector","type":"sink","version":"1.0.1"},
        {"class":"org.apache.kafka.connect.file.FileStreamSourceConnector","type":"source","version":"1.0.1"}
    ]

The Connect node has the Debezium `MySqlConnector` connector plugin available.

## Congratulations

You have now successfully executed the second step in this scenario. 

You have successfully deployed Kafka Connect node and configured it to contain Debezium.

In next step of this scenario we will finish deployment by creating a connection between database source and Kafka Connect.
