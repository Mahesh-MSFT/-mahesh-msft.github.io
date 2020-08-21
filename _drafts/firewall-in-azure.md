---
title: "Choosing right Firewall service in Azure"
date: 2020-08-20
categories:
  - blog
tags:
  - Azure Firewall
  - Azure Application Gateway
  - Azure Front Door
  - NVA
---

Azure has multiple services providing firewall capabilities. Features provided by these services are well documented. Many enterprises have separate security and development teams. It is common for enterprise security teams to mandate an Azure service that provides firewall capability as part of architecture for an application. This post will help enterprise security teams make an informed decision on choosing right firewall service for applications running on Azure.

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

An user request traversal for Azure Firewall, Azure Application Gateway and Azure Front Door is described below.

### Azure Firewall traversal

Azure Firewall traversal consists of following.

1. User request arrives at a public IP address.
1. Azure Firewall has a NAT rule to route the incoming request to private address of Azure Internal Load Balancer associated with Service controller in AKS.
1. An Azure User Defined Route (UDR) is created with Azure Firewall as "Next Hop". This UDR is associated with subnet containing AKS cluster.
1. Azure Firewall has Network and Application rules created to allow egress traffic originating from AKS cluster.

### Azure Application Gateway traversal

Azure Application Gateway traversal consists of following.

1. User request arrives at a public IP address which is configured as a front-end IP address for Azure Application Gateway.
1. Azure Application Gateway is configured with *Prevention* as Firewall mode.
1. Azure Application Gateway has a HTTP probe created for Azure Internal Load Balancer inside AKS subnet.

### Azure Front Door traversal

Azure Front Door traversal consists of following.

1. User request arrives at a public IP address which is configured as a backend address for Azure Front Door.
1. XXXX.
1. Azure Front Door has a WAF Policy created with *Prevention* mode.
1. WAF Policy associated with Azure Front Door has Azure Managed Rule Set assigned to it.
1. WAF Policy with Azure Managed Rule Set is applied to the Azure Front Door front endpoint.

## SQL Injection vulnerability

Sample application is created with known SQL injection vulnerability. Application has components as described below.

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

Java Servlet accepts the FORM data. It extracts the username and password fields. It further connects with MySQL database. Using username and password, it issues a query against table.

```java
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        boolean success = false;
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        // Password =  ' or '1'='1
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