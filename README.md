# Step-by-Step Guide to Add Windows Server to ELK #

1. Install Winlogbeat on Windows Server
	Visit the official Elastic website:
	https://www.elastic.co/downloads/beats/winlogbeat
      Download and extract the ZIP file to C:\Program Files\Winlogbeat.

2. Open PowerShell as Administrator
	cd "C:\Program Files\Winlogbeat"

3. Configure Winlogbeat
	Edit the winlogbeat.yml Configuration File
	notepad C:\Program Files\Winlogbeat\winlogbeat.yml

4. Modify the Output Section
	If sending logs directly to Elasticsearch
		output.elasticsearch:
	  	hosts: ["http://<ELK-SERVER-IP>:9200"]
  		username: "elastic"
  		password: "yourpassword"
	If using Logstash (recommended for processing logs)
		output.logstash:
		hosts: ["<ELK-SERVER-IP>:5044"]


5. Enable Windows Event Logs (Optional)
		winlogbeat.event_logs:
  		- name: Application
  		- name: Security
  		- name: System

6. Install and Start the Winlogbeat Service
	Install Winlogbeat as a Windows Service
		.\install-service-winlogbeat.ps1
	Start Winlogbeat
		Start-Service winlogbeat
	Verify the Service Status
		Get-Service winlogbeat

7. Configure Logstash to Process Logs (If Used)
	SSH into your ELK server and create a config file:
		sudo nano /etc/logstash/conf.d/winlogbeat.conf

8. Add the following Logstash configuration:
		input {
  beats {
    port => 5044
  }
}

filter {
  mutate {
    add_field => { "source" => "windows-server" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "winlogbeat-%{+YYYY.MM.dd}"
  }
}


9. Restart Logstash
		sudo systemctl restart logstash


10. Verify Data in Kibana
		Open Kibana: http://<ELK-SERVER-IP>:5601
		Navigate to Stack Management → Index Patterns.
		Create a new pattern for winlogbeat-*.
		Open Discover and check incoming logs.	
