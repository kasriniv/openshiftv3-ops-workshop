# RH SSO Installation Quickstart

These instructions are a cheat sheet for RHSSO install to be used with 3scale for SSO, OAuth etc usecases.

## Step 1: Install Java 1.8 and Verify

```
sudo bash
cd /opt/

wget -c --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz

tar xzf jdk-8u131-linux-x64.tar.gz

cd /opt/jdk1.8.0_131

alternatives --install /usr/bin/java java /opt/jdk1.8.0_131/bin/java 1
alternatives --config java
1

sudo alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_131/bin/jar 2
sudo alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_131/bin/javac 2
sudo alternatives --install /usr/bin/javaws javaws /opt/jdk1.8.0_131/bin/javaws 2
sudo alternatives --set jar /opt/jdk1.8.0_131/bin/jar
sudo  alternatives --set javac /opt/jdk1.8.0_131/bin/javac

```

Setup Environment Variables

```
export JAVA_HOME=/opt/jdk1.8.0_131
export JRE_HOME=/opt/jdk1.8.0_131/jre
export PATH=$PATH:/opt/jdk1.8.0_131/bin:/opt/jdk1.8.0_131/jre/bin
```

Verify

```
java -version

```

## Step 2: Move files to your AWS instance

```
scp -i ./your-pem-file your local location/jboss-image-streams.json ec2-user@<AWS Public DNS>:/home/ec2-user
scp -i ./your-pem-file your local location/sso70-mysql-persistent.json ec2-user@<AWS Public DNS>:/home/ec2-user
```

## Step 3: Set up keystores

Set up SSL keystore

```keytool -genkeypair -alias keystore -storetype jks -keyalg RSA -keysize 2048 -keypass password1 -keystore keystore.jks -storepass password1 -dname "CN=SSL-Keystore,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v```

Set up Jgroups Keystore

```keytool -genkeypair -alias jgroups -storetype jceks -keyalg RSA -keysize 2048 -keypass password1 -keystore jgroups.jceks -storepass password1 -dname "CN=JGROUPS,OU=Sales,O=Systems Inc,L=Raleigh,ST=NC,C=US" -validity 730 -v```

# Step 4: Create RHSSO project on Openshift and install

```
oc login -u developer (or any account that has developer privs)
oc new-project rhsso

oc create secret generic sso-app-secret --from-file=keystore.jks --from-file=jgroups.jceks
oc create serviceaccount sso-service-account
oc policy add-role-to-user view system:serviceaccount:rhsso:sso-service-account -n rhsso
oc secret add sa/sso-service-account secret/sso-app-secret
```

Login as admin to install image streams

```
sudo oc login -u system:admin

sudo oc create -n openshift -f jboss-image-streams.json
sudo oc process -f sso71-mysql.json -v HTTPS_NAME=keystore -v HTTPS_PASSWORD=password1 | sudo oc create -n rhsso -f -
```

## Step 5: Routes and RHSSO login

* Browse to your application.  Go to https://secure-sso-rhsso.${YOUR_WILDCARD_URL}/auth/

* Change the 'sso' openshift deployment

* Choose Administration Console

* Login as admin/admin

* Under Realm Settings -> Login, set Require SSL to none

(If you cant login in, go to the sso deployment and change SSO_ADMIN_USERNAME and SSO_ADMIN_PASSWORD to admin)

* Create a realm <myrealm>. 
	* Under Realm Settings -> Login, set Require SSL to none
	
You are now ready to move on to configuring RHSSO with 3scale ! :tada:
