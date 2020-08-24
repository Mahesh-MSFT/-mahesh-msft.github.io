---
title: "Choosing right Firewall service in Azure"
date: 2020-08-20
categories:
  - blog
tags:
  - Azure Firewall
  - Azure Application Gateway
  - Azure Front Door
---

Azure has multiple services providing firewall capabilities. Features provided by these services are well documented. However, seeing these capabilities in action can help  evaluate each service before making a decision. This post will use a SQL injection attack to showcase firewall capabilities.

Below are three key Azure services that provide firewall services.

1. Azure Firewall
1. Azure Application Gateway
1. Azure Front Door

High-level features offered by Azure Firewall, Azure Application Gateway and Azure Front Door are discussed below.

## Azure Firewall

Azure firewall is a managed service. It provides high-availability. It uses built-in threat-intelligence based filtering of network traffic. It has unrestricted scalability to match growth in network traffic. It enables enterprises to author and apply policies for common tasks such as restricting access, preventing data exfiltration, etc.

## Azure Application Gateway

Azure Application Gateway is a layer 7 load balancer. It has built-in automatic failover and no downtime maintenance capabilities. Azure Application Gateway provides Web Application Firewall (WAF) services for ingress HTTP traffic. It provides SSL offloading capabilities which helps to removes decryption processing burden on web servers. Azure Application Gateway can route traffic based on URL Paths and supports cookie based session affinity.

## Azure Front Door

Azure Front Door provides secure entry point in Azure. It is a global HTTP load balancer with seamless failover capability. It helps prevent application and network layer attacks at the edge locations. Azure Front Door supports multiple configurations (e.g. round-robin, weighted round-robin, active/standby, etc.) for load balancing rules. It provides insights into the traffic through metrics such as client requests, latency, performance, etc. It provides optimized routing path between users and applications.

## Sample application

This post will cover creating a Java application with a known SQL injection vulnerability. This application will be deployed as a service controller on Azure Kubernetes Service (AKS). AKS Service controller will be deployed behind an internal load balancer with a private IP address. Application architecture will look like below.

![design](/assets/aks/firewallpost.jpg)

An user request flow for Azure Firewall, Azure Application Gateway and Azure Front Door is described below.

### Azure Firewall request flow

Azure Firewall request flow consists of following.

1. User request arrives at a public IP address.
1. Azure Firewall has a NAT rule to route the incoming request to private address of Azure Internal Load Balancer associated with Service controller in AKS.
1. An Azure User Defined Route (UDR) is created with Azure Firewall as "Next Hop". This UDR is associated with subnet containing AKS cluster.
1. Azure Firewall has Network and Application rules created to allow egress traffic originating from AKS cluster.

### Azure Application Gateway request flow

Azure Application Gateway request flow consists of following.

1. User request arrives at a public IP address which is configured as a front-end IP address for Azure Application Gateway.
1. Azure Application Gateway is configured with *Prevention* as Firewall mode.
1. Azure Application Gateway has a HTTP probe created for Azure Internal Load Balancer inside AKS subnet.

### Azure Front Door request flow

Azure Front Door request flow consists of following.

1. User request arrives at a public IP address which is configured as a backend address for Azure Front Door.
1. Azure Front Door uses public IP or any publically routable DNS address as it's backend. In this example, Azure Front Door is using Azure Firewall's public IP as a backend.
1. Azure Front Door has a WAF Policy created with *Prevention* mode.
1. WAF Policy associated with Azure Front Door has Azure Managed Rule Set assigned to it.
1. WAF Policy with Azure Managed Rule Set is applied to the Azure Front Door front endpoint.

## SQL Injection vulnerability

Sample application is written using Java Servlet. It has a known SQL injection vulnerability. Application has components as described below.

> This post has used Java Servlet with SQL injection for demo purpose only. This does not imply that Java Servlet has this vulnerability. Java Servlet is a mature platform and has platform features to prevent SQL injection.

### HTML Page

HTML page is configured to POST form data to a Java Servlet

```HTML
<form name="frmLogin" method="POST" action="LoginServlet">
        <table>
            <tr>
                <td>Username</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>Password</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td colspan="2"><button type="submit">Login</button></td>
            </tr>
        </table>
    </form>
```

### Servlet

Java Servlet accepts the FORM data. It extracts the username and password fields as shown below.

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        boolean success = false;
        String username = request.getParameter("username");
        String password = request.getParameter("password");
```

It further connects with MySQL database. A query using username and password is made against database table.

```java
        String query = "select * from tblUser where username='" + username + "' and password = '" + password + "'";
        Connection conn = null;
        Statement stmt = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://mysqlsvc:3306/userDB", "root", "root");
            stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery(query);
            if (rs.next()) {
                // Login Successful if match is found
                success = true;
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                stmt.close();
                conn.close();
            } catch (Exception e) {}
        }
        if (success) {
            response.sendRedirect("home.html");
        } else {
            response.sendRedirect("unauth.html");
        }
    }
```

If query fetches any record from table then login is considered as successful. If login is successful then users are navigated to *home* page else they will be navigated to *unauth* page.

> MySQL database is also running as a Service controller in AKS. Java Servlet Connectionstring uses the Service controller name for MySQL connectivity.

### Dockerfile

Java Servlet as well as MySQL database are *containerized* using following dockerfiles respectively.

```dockerfile
FROM maven:3.5-jdk-8 as BUILD
  
COPY src /usr/src/sji/src
COPY pom.xml /usr/src/sji
RUN mvn -f /usr/src/sji/pom.xml clean package
RUN mvn -f /usr/src/sji/pom.xml install package

FROM tomcat:jdk8-openjdk
WORKDIR /

COPY --from=build /usr/src/sji/target/jsi.war /usr/local/tomcat/webapps/jsi.war

EXPOSE 80

CMD ["catalina.sh", "run"]
```

```dockerfile
# Derived from mysql base image
FROM mysql:latest

# Add a database
ENV MYSQL_DATABASE userDB
ENV MYSQL_ROOT_PASSWORD root

# Add the content of the sql-scripts/ directory to image
# All scripts in docker-entrypoint-initdb.d/ are automatically
# executed during container startup
COPY ./sql-scripts/ /docker-entrypoint-initdb.d/
```

### Kubernetes manifests

Java Servlet and MySQL are deployed to AKS using following manifests.

```yml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: <YOUR-REPO>.azurecr.io/mysql:latest
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
                secretKeyRef:
                  name: mysqlsecret
                  key: mysqlpassword
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
      - name: mysql-initdb
        configMap:
          name: mysql-init-config

---
apiVersion: v1
kind: Service
metadata:
  name: mysqlsvc
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jsi
spec:
  selector:
    matchLabels:
      app: jsi
  template:
    metadata:
      labels:
        app: jsi
    spec:
      containers:
      - name: jsi
        image: <YOUR-REPO>.azurecr.io/jsi:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: jsi
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  selector:
    app: jsi
  type: LoadBalancer
  loadBalancerIP: 10.42.1.100
  ports:
  - port: 80
    targetPort: 8080
```

## Testing with Azure Firewall

Azure Firewall has a NAT rule configured that routes incoming requests on its public IP address to private IP address of Azure internal load balancer in AKS subnet.

```azurecli
az network firewall nat-rule create -g $RG -f $FWNAME `
    --collection-name 'aksfwnatr' -n 'inboundtcp' --protocols 'TCP' --source-addresses '*' `
    --action Dnat  --priority 100 `
    --destination-addresses $FWPUBLIC_IP --destination-ports 80 `
    --translated-address '10.42.1.100' --translated-port 80
```

Navigation to Azure Firewall's public IP address loads application as show below.

![fwload](/assets/aks/aksload.png)

User enters any username and following password.

```sql
 ' or '1'='1
```

This password gets *injected* into SQL query as shown below.

```sql
select * from tblUser where username='dummy' and password = '' or '1'='1'
```

SQL query gets evaluated to *true* because of the `1=1` condition. User is promptly navigated to *home* page as shown below.

![afwhome](/assets/aks/afwhome.png).

This outcome is expected. Azure Firewall does not provide Web Application Firewall (WAF) capability. Azure Firewall provides inbound protection for non-HTTP/S protocols. Azure Firewall should be used to protecting incoming traffic over RDP, FTP and SSH protocols.

## Testing with Azure Application Gateway

Azure Application Gateway is configured with *Prevention* as firewall mode.

```azurecli
az network application-gateway waf-config set `
  --enabled true `
  --gateway-name $AGW_NAME `
  --resource-group $RG `
  --firewall-mode Prevention `
  --rule-set-version 3.0
```

Azure Application Gateway has a HTTP probe created for the private IP address of the internal load balancer inside AKS subnet.

```azurecli
az network application-gateway probe create `
    -g $RG `
    --gateway-name $AGW_NAME `
    -n defaultprobe-Http `
    --protocol http `
    --host <PRIVATE IP OF INTERNAL LOAD BALANCER in AKS> `
    --timeout 30 `
    --path /jsi
```

Navigating to public IP address of the Azure Application Gateway loads the application home page.

However, if user enters password `' or '1'='1` that initiates SQL injection attack, an error page is shown as below.

![agwerror](/assets/aks/agwerror.png)

Azure Application Gateway has intercepted the SQL injection. Instead of navigating user to *Home* page, an error page is shown to user.

## Testing with Azure Front Door

Azure Front Door is configured with Azure Firewall's public IP as its backend address. It is not necessary to use Azure Firewall. Azure Front Door can use any public IP or publically routable DNS address as its backend.

```azurecli
az network front-door create `
    --resource-group $RG `
    --name $AFD_NAME `
    --backend-address $FWPUBLIC_IP `
    --path /jsi
```

A WAF policy is created and configured in *Prevention* mode.

```azurecli
az network front-door waf-policy create `
    --resource-group $RG `
    --name $AFD_POLICY_NAME `
    --mode "Prevention" `
    --disabled False
```

Azure Front Door has *managed rules* which includes rule to protect against SQL injection. Azure Front Door Managed Rules are added in WAF policy.

```azurecli
az network front-door waf-policy managed-rules add `
    --policy-name $AFD_POLICY_NAME `
    --resource-group $RG `
    --type "DefaultRuleSet" `
    --version "1.0"
```

Finally, WAF policy is applied to the default front end endpoint of Azure Front Door.

```azurecli
az network front-door update `
    --resource-group $RG `
    --name $AFD_NAME `
    --set FrontendEndpoints[0].WebApplicationFirewallPolicyLink.id=/subscriptions/$SUB_ID/resourcegroups/$RG/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/$AFD_POLICY_NAME 
```

When a user navigates to Azure Front Door front address (e.g [YOUR-AFD-NAME].azurefd.net) application login page is shown.

If user enters password `' or '1'='1` that initiates SQL injection attack, a request blocked page is shown as below.

![afdblocked](/assets/aks/afdblocked.png)

Azure Front Door has blocked the request which had SQL injection script in it.

## Summary

Azure Firewall provides firewall capabilities for non-HTTP ingress traffic. For HTTP ingress traffic use Azure Application Gateway or Azure Front Door. Azure Front Door needs a public IP or a publically routable DNS address as it's backend. Azure Application Gateway can use private IP address as it's backend.

## Additional resources

* [SQL Injection in Java and How to Easily Prevent it
](https://www.journaldev.com/34028/sql-injection-in-java#java-sql-injection-example)
* [Azure Firewall FAQ](https://docs.microsoft.com/en-us/azure/firewall/firewall-faq)
* [Frequently asked questions about Application Gateway](https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-faq)
* [Frequently asked questions for Azure Front Door](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-faq)