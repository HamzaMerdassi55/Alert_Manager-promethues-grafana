## Monitoring and Alerting for Spring Boot with Prometheus, prometheus_alerte-mananger, Grafana and Gmail,slack,pgdurty

# Step0 : configure env

- Download vm ( image Server ubuntu 22.04) link : ' https://ubuntu.com/download/server/choose '

- installed you image in vmware workstation

- instal wsl in you local Machine (physique) 

- open you server ubuntu and install open-ssh and client-server ssh 
  you can find all the link to install here : ' https://ubuntu.com/server/docs/service-openssh '
=> do this in wsl to 


- find your ip address in server ubuntu with this commande : ip a or ifconfig(sudo apt-get install net-tools
)

now ping your adress ip server ubuntu in wsl, when evreything is ok 

- conection  wsl with server ubuntu vmware  : ssh@ip_adress_serverubunt

# Step1 build infrastrucure

1. install docker => ' https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04  '

2. install docker-compose : 
sudo apt update
sudo apt install docker-compose

3. install potainer.io : 
- docker volume create portainer_data
   - docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
    - docker run -d -p 9000:9000 \

4. go to your Browser click in your urk and add your address_ip_server_ubuntu:9000
=> now your potainer is run 

5. go to stack in container and add docker-compose name and create the code yml
i add the file docker-compose.yml in the folder chek hem ans just vopy and pâste in your portainer 
the click in action create

6. now your portainer and all image is run 

# Step2 create prometheus

1. in wsl go to etc( cd /etc ) and create folder promtheus
2. Create a new file named prometheus.yml and add the following contents:

global:
scrape_interval: 15s
evaluation_interval: 15s
scrape_configs:
- job_name: 'spring-boot-prometheus'
metrics_path: '/actuator/prometheus'
static_configs:
- targets: ['ipaddressserver:8080'] 

3. To configure alerting rules, we need to define them in Prometheus. Open
the prometheus.yml file and add the following contents:

# Rule files
rule_files:
  - 'alert-rules.yml'

# Alerting configurations
alerting:
  alertmanagers:
    - scheme: http
    - static_configs:
        - targets: ['alertmanager:9093']

- save and close nano prometheus.yml

# Step3 Create Alerting Rules 

1. in the same folder promotheus , create alert-rules.yml file that has some rules to guide Prometheus on when to raise
alerts. We’re going to create a set of instructions to let Prometheus know when a critical
situation requires our attention.

groups:
  - name: lab-rules
    rules:
     # Triggers a critical alert if a server is down for more than 1 minute.
     - alert: ServerDown
       expr: up < 1
       for: 1m
       labels:
         severity: critical
       annotations:
         summary: "Server {{ $labels.instance }} down"
         description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

2. Configure Alertmanager

Having set up rules for the AlertManager, the natural question arises: how do we ensurethese critical alerts reach the right people at the right time?
The answer lies in configuring routes and receivers within the alertmanager.yml file.
Here's a breakdown of the process:
go to folder alert-manager -> config Create and open the alertmanager.yml file to configure how alerts will be directed and who will receive them.
copy and paste that

global:
resolve_timeout: 1m
slack_api_url: 'https://hooks.slack.com/services/!!!PUT_YOURS!!!'
route:
receiver: lab-alert-manager
repeat_interval: 1m
receivers:
- name: 'lab-alert-manager'
telegram_configs:
- bot_token: lab_token
api_url: https://api.telegram.org
chat_id: -12345678
parse_mode: ''
send_resolved: true
email_configs:
- to: 'lab.inbox@gmail.com'
from: 'lab.outbox@gmail.com'
smarthost: 'smtp.gmail.com:587'
auth_username: 'lab.outbox@gmail.com'
auth_identity: 'lab.outbox@gmail.com'
auth_password: 'AppPassword'
send_resolved: true
slack_configs:
- channel: '#monitoring-infrastructure'
send_resolved: true

# Step 3: Configure Spring Boot Application
1. download springboot application so go to https://start.spring.io/ chose maven and complete procees then download and open with your editor code (inteliji or eclipse)
2. Add the required dependencies into your project. Open your pom.xml file and insert the
following dependencies:


<dependency>
<groupId>io.micrometer</groupId>
<artifactId>micrometer-registry-prometheus</artifactId>
<scope>runtime</scope>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>


3. Next, open your Spring Boot application’s application.properties file and add the
following lines to enable the required management endpoints:

management.endpoints.web.exposure.include= *
management.endpoint.prometheus.enabled= true
management.endpoint.health.show-details= always
management.endpoint.metrics.enabled= true
management.prometheus.metrics.export.enabled= true

# step 4 : Visualizing Metrics and Creating Dashboards with Grafana

With Docker, Prometheus, and Grafana set up, we can now start visualizing the metrics
collected from our Spring Boot application and create customized dashboards in Grafana.

1. Run Spring Boot Application

2. Access Grafana Web Interface

Open your web browser and navigate to ipaddressserver:3000 to access the Grafana
web interface. You will be prompted to log in with the default credentials (username: admin,
password: admin). After logging in, you will be prompted to change the password.

3. Once you have logged in, click on the “Connections” icon in the left-hand menu, then click
on “Data Sources”. Click on the “Add data source” button and select “Prometheus” as the
data source type. Configure the URL to ipaddressserver:9090 and click on the "Save
& Test" button. If the connection is successful, you will see a green notification confirming
that the data source is working.

4. Step 4: Create a Dashboard

Now that we have configured Prometheus as the data source, it’s time to create a new dashboard for visualizing the metrics. Grafana offers a wide range of visualization options,
including charts, tables, and gauges, allowing you to create custom dashboards. You can discover an assortment of diverse dashboards, curated by countless users, within the Grafana Dashboards repository. However, for our purpose, we’ll tap into the Spring Boot 2.1 Statistics dashboard (ID: 10280).
To initiate the process, navigate to the left menu and click on the ‘Dashboards’ icon. Then,
proceed by selecting ‘New -> Import.’ Within this interface, you can import dashboards
using either a dashboard URL or ID, subsequently adapting them to showcase the metrics
relevant to your context. As we’re focusing on the Spring Boot 2.1 Statistics dashboard, input
the ID 10280 and import it. When prompted, associate the imported dashboard with the
Prometheus data source.

5. / Fire an Alert
Temporarily stop your Spring Boot Application. After a minute, observe the triggering of the
ServerDown alert. Notifications will be dıspatched through the specified channels
if *_configs are defined correctly.

# Step 5 : Gmail: Create & use app passwords
Important: To create an app password, you need 2-Step Verification on your Google Account.
If you use 2-Step-Verification and get a "password incorrect" error when you sign in, you
can try to use an app password.
1. Go to your Google Account.
2. Select Security.
3. Under "Signing in to Google," select 2-Step Verification.
4. At the bottom of the page, select App passwords.
5. Enter a name that helps you remember where you’ll use the app password.
6. Select Generate.
7. To enter the app password, follow the instructions on your screen. The app password is
the 16-character code that generates on your device.
8. Select Done.

# Step 6 : set up Slack alerts
- To set up alerting in your Slack workspace, you’re going to need a Slack API URL. Go
to Slack -> Administration -> Manage apps. 
- In the Manage apps directory, search for Incoming WebHooks and add it to your Slack
workspace.
Next, specify in which channel you’d like to receive notifications from Alertmanager.
(Create #monitoring-infrastructure channel.) After you confirm and add Incoming
WebHooks integration, webhook URL (which is your Slack API URL) is displayed. Copy it.

# Step 7 : set up PagerDuty alerts
1. PagerDuty is one of the most well-known incident response platforms for IT departments.
To set up alerting through PagerDuty, you need to create an account there. (PagerDuty is
a paid service, but you can always do a 14-day free trial.) Once you’re logged in, go
to Configuration -> Services -> + New Service.
2. Choose Prometheus from the Integration types list and give the service a name like
‘Prometheus Alertmanager’. (You can also customize the incident settings, but choose
the default setup.) Then click save.
3. You’ll need to update the content of your alertmanager.yml. It should look like the
example below, but use your own service_key (integration key from
PagerDuty). Pagerduty_url should stay the same and should be set
to https://events.pagerduty.com/v2/enqueue. Save and restart the Alertmanager.
(Reload configuration by sending POST request to /-/reload endpoint curl -X POST
http://localhost:9093/-/reload)

 global:
resolve_timeout: 1m
pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'
route:
receiver: 'pagerduty-notifications'
receivers:
- name: 'pagerduty-notifications'
pagerduty_configs:
- service_key: 0c1cc665a594419b6d215e81f4e38f7
send_resolved: true


4. Stop one of your instances. After a couple of minutes, alert notifications should be
displayed in PagerDuty.




=> you can create email professional with ovh 
https://www.ovhcloud.com/fr/mail/?fbclid=IwAR3p76fwNPKE3xt5nXAkqtLtd-7CfxUUBpitKq46wUO0n_3uPdXhHsDnkjM