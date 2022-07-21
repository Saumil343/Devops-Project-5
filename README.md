# Devops-Project-5
Project Life Cycle:
![devops project 5 drawio](https://user-images.githubusercontent.com/53990452/180130006-8fbf79ce-c3f3-427b-a149-f3e7784ead03.png)
- <b> Tool's Used: </b>
- Terraform
- Jenkins
- Ansible
- Github
- Docker Containers 
- Aws Load Balancer


- <b> Steps Followed </b>

- <b> Project Descrption </b>
- The project uses the concept of Infrastructre as code and uses terraform for creating a architecture on the AWS, Also I have used a jenkins pipeline to execute Ansible Playbooks over two webservers which i created using terraform, after that the ansible playbook also pulls the latest code from the github and creates a docker image and also runs it on port 3100, Later on i have created a Application Load Balancer and target group with terraform which also adds this two web server to the target group of the load balancer which health checks the port 3100 over web server
