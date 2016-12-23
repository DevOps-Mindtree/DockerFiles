Pull command: docker pull devopsmt/sonar6.0-mysql5.6

Run command: docker run -d -p 3306:3306 -p 9000:9000 --name hg-sonar devopsmt/sonar6.0-mysql5.6

Build Image Command: docker build -t devopsmt/sonar6.0-mysql5.6 .
