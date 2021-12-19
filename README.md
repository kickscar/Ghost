<p align="center"><a href="https://ghost.org/"><img src="https://user-images.githubusercontent.com/120485/43974508-b64b2fe8-9cd2-11e8-8e58-707254b8817c.png" width="140px" alt="Ghost" /></a><br/><br/><span>Copyright (c) 2013-2021 Ghost Foundation - Released under the <a href='LICENSE'>MIT license</a><br/>Ghost and the Ghost Logo are trademarks of Ghost Foundation Ltd.<br/>Please see our <a href='https://ghost.org/trademark/'>trademark policy</a> for info on acceptable usage.</span><br/><br/><br/></p>


## How to Install Ghost

### Prerequisites
1.	Node.js
	- Recommended: 14.x(V14 Fermium LTS)
	- Supported: 12.x (Node v12 Erbium LTS), 16.x (v16 Gallium LTS)
	- Recommended Ideally: nvm(Node Version Manager)* Environments
 
2.	Linux
	- Recommended: Ubuntu(16.04, 18.04, 20.04 LTS)
	- CentOS(6+)*
		+ development mode only: [Local Installation with ghost-cli](https://ghost.org/docs/install/local/)
		+ production mode available: [Source Installation*](https://ghost.org/docs/install/source/)(ghost-cli not need)

3.	Reverse Proxy Setup
	- Apache: mod_proxy
	- Nginx*: Basics 

### Create Database ghost(MairaDB)

```sh
mysql -p
Enter password:

MariaDB [(none)]> create database <dbname>
MariaDB [(none)]> create user <dbuser>@localhost identified by '<dbpassword>'
MariaDB [(none)]> grant all privileges on <dbname>.* to ghost@localhost
MariaDB [(none)]> flush privileges
```

### Create Ghost Git Repository Clone

1.	Installation: ghost blog full package(core, admin, theme)

	```sh
	# 서브모듈을 포함하여 Ghost 소스를 clone 한다. 
	git clone --recurse-submodules git@github.com:TryGhost/Ghost
	```

2.	Configuration I: Ghost Core

	```sh
	# 작업 디렉토리 이동
	cd Ghost
	
	# origin remote를 upstream으로 변경: upstream <-> TryGhost/Ghost
	git remote rename origin upstream
	
	# origin remote 추가: origin <-> <GithubUsername>/Ghost
	git remote add origin git@github.com:<GithubUsername>/Ghost.git

	# Guthub에 Ghost 레포지토리 생성: gh(Github CLI) 설치 필요 (https://cli.github.com/manual/gh)
	gh repo create Ghost --public
	
	# push
	git push -u origin main
	```
	
3.	Configuration II: Ghost Admin

	```sh
	# 작업 디렉토리 이동
	cd core/client
	
	# origin remote를 upstream으로 변경: upstream <-> TryGhost/Admin
	git remote rename origin upstream
	
	# origin remote 추가: origin <-> <GithubUsername>/Ghost-Admin
	git remote add origin git@github.com:<GithubUsername>/Ghost-Admin.git
	
	# Guthub에 Ghost 레포지토리 생성: gh(Github CLI) 설치 필요 (https://cli.github.com/manual/gh)
	gh repo create Ghost-Admin --public
	
	# push
	git push -u origin main	
	```

4.	Up to date

	```sh
	git submodule sync
	git submodule update
		
	git diff --exit-code --quiet --ignore-submodules=untracked
	git checkout main
		
	git pull upstream main
	git pull origin main
	
	git push
	```
	
### Installation &amp; Configuration
1.	Installation(production mode)

	```sh
	# 설치 디렉토리에서 install script 실행
	npm install --production
	
	```

3.	Configuring
	-	blog url 설정: core/shared/config/defaults.json

		```json
		{
			"url": "http(s)://<BlogHostDomain or BlogHostIp>",
			"server": {
				"host": "127.0.0.1",
				"port": 2368,
				"shutdownTimeout": 60000
			},
			"admin": {
				"redirects": true
			},
			
			[...skip...]
		}
		```

	-	database 연결 설정: core/shared/config/env/config.production.json
		
		```json
		{
			"database": {
				"client": "mysql",
				"connection": {
					"host"     : "<dbhost>",
					"user"     : "<dbuser>",
					"password" : "<dbpassword>",
					"database" : "<dbname>"
				}
			},

			[...skip...]
		}
		```

### Start Blog

1.	start

	```sh
	npm start --production

	# or

	node index.js

	# 다음과 같은 내용이 출력
	[2021-12-15 15:53:38] INFO Ghost is running in production...
	[2021-12-15 15:53:38] INFO Your site is now available on http://blog.kickscar.me/
	[2021-12-15 15:53:38] INFO Ctrl+C to shut down
	[2021-12-15 15:53:38] INFO Ghost server started in 0.274s
	[2021-12-15 15:53:39] INFO Database is in a ready state.
	[2021-12-15 15:53:39] INFO Ghost database ready in 0.414s
	[2021-12-15 15:53:40] INFO Ghost booted in 1.487s
	[2021-12-15 15:53:40] INFO Adding offloaded job to the queue
	[2021-12-15 15:53:40] INFO Scheduling job update-check at 11 50 15 * * *. Next run on: Thu Dec 16 2021 15:50:11 GMT+0900 (대한민국 표준시)
	[2021-12-15 15:53:40] INFO Ghost URL Service Ready in 1.806s

	```

2.	test

	```sh
	curl -I localhost:2368
	
	# 응답 헤더
	HTTP/1.1 200 OK
	X-Powered-By: Express
	Cache-Control: public, max-age=0
	Content-Type: text/html; charset=utf-8
	Content-Length: 24078
	ETag: W/"5e0e-5Auo8SMsN0ylysove5kf06j/l5M"
	Vary: Accept-Encoding
	Date: Thu, 16 Dec 2021 10:09:02 GMT
	Connection: keep-alive
	Keep-Alive: timeout=5

	[2021-12-16 19:09:02] INFO "HEAD /" 200 46ms
	```

### Configuring Nginx to Proxy Requests to Ghost
1.	/etc/nginx/conf.d/ghost.conf

	```sh
	server {
		listen       80;
		server_name  <YourBlogDoamin>;

		location / {
			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-Proto $scheme;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

			proxy_pass http://127.0.0.1:2368;
		}
	}	
	```

2.	test configuration

	```sh
	nginx -T
	
	ginx: the configuration file /etc/nginx/nginx.conf syntax is ok
	nginx: configuration file /etc/nginx/nginx.conf test is successful
	configuration file /etc/nginx/nginx.conf:
	
	[...skip...]
	
	```

5.	server reload

	```sh
	nginx -s reload
	
	```
	
6.	test

	```sh
	curl -I <YourBlogDoamin>
	
	HTTP/1.1 200 OK
	Server: nginx/1.21.4
	Date: Fri, 17 Dec 2021 10:47:09 GMT
	Content-Type: text/html; charset=utf-8
	Content-Length: 24078
	Connection: keep-alive
	X-Powered-By: Express
	Cache-Control: public, max-age=0
	ETag: W/"5e0e-GmzbIOQPWhHhHXUCwHacZNoMhFU"
	Vary: Accept-Encoding

	[2021-12-17 19:47:09] INFO "HEAD /" 200 38ms
	
	```

### Create a New User: Running Ghost as a Separate User


### Running Ghost as a System Service
1.	systemd unit file: ghost.service
	
	```
	[Unit]
	Description=Ghost
	After=network.target

	[Service]
	Type=simple

	WorkingDirectory=<YoutGostInstallDirectory>
	User=ghost
	Group=ghost

	ExecStart=<YourNodexecutable> start --production
	ExecStop=<YourNPMExecutable> stop --production
	Restart=always
	SyslogIdentifier=Ghost

	[Install]
	WantedBy=multi-user.target
	```
	
3.	1
