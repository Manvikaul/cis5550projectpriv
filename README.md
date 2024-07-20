# Cloud-based Search Engine
### Manvi Kaul, Kevin Zhang, Spencer Mateega, and Mahika Vajpeyi
 
## Run Instructions
 
This project involves the development of a robust cloud-based search engine. Our team focused on implementing key components, including an efficient indexer, a PageRank algorithm for ranking web pages, and an intuitive front-end interface. We integrated various features such as a blacklist to filter low-quality pages and dynamic table management for optimized performance. The solution was deployed on Amazon EC2, ensuring scalability and reliability. Key challenges tackled included handling large volumes of data, managing crawler operations, and optimizing cloud resources.
 
### Running the crawler on one instance
 
Boot up one instance on EC2, clone this repo, and cd into the Crawler folder.
 
First run the following command in the instance’s terminal to compile the code:
```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src -d bin $(find src -name '*.java')
```
Then, open up 10 new terminals. Run each of the following lines in seperate terminals:
 
```sh
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Coordinator 8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8001 worker1 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8002 worker2 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8003 worker3 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8004 worker4 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Coordinator 9000 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9001 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9002 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9003 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9004 localhost:9000
```
 
This will start a KVS coordinator, 4 KVS workers, a Flame coordinator, and 4 Flame workers. Go to the 8000 port on the instance’s public IP to view the KVS coordinator and port 9000 to view the Flame coordinator.
 
Finally, run the following commands in a new terminal to execute the crawler:
 
```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src src/cis5550/*/*.java
jar cf crawler.jar classes/cis5550/jobs/Crawler.class
nohup java -cp bin:lib/kvs.jar:lib/webserver.jar:lib/flame.jar:lib/json-20140107.jar cis5550.crawling.runCrawl
```
 
### Running the crawler on multiple instances
 
We used 5 EC2 instances to run the crawler.
 
Boot up 5 instances. On each instance clone this repo and cd into the Crawler folder.
 
First run the following command in each instance’s terminal to compile the code:
```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src -d bin $(find src -name '*.java')
```
 
On the first instance, run the following commands (each line should be in a new terminal):

```sh
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Coordinator 8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Coordinator 9000 localhost:8000
```

Locate this first instance’s public IP address in the EC2 web-portal. On the second instance, head to Crawler.java (src>cis5550>jobs>Crawler.java). Scroll down to line 261 and update the line `KVSClient kvs = new KVSClient("localhost:8000");` by replacing localhost with the IP address of the first instance. Then run the following commands (again, each line should be in a new terminal):

```sh
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8001 worker1 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9001 localhost:9000
```

Repeat these steps on the third, fourth, and fifth instances to set up the remaining workers.

Finally, run the following commands in a new terminal on the first instance (the instance running the coordinators) to execute the crawler:
 
```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src src/cis5550/*/*.java
jar cf crawler.jar classes/cis5550/jobs/Crawler.class
nohup java -cp bin:lib/kvs.jar:lib/webserver.jar:lib/flame.jar:lib/json-20140107.jar cis5550.crawling.runCrawl
```

### Running the indexer 

We will describe how to run the remaining three components on one instance, but to run the code across multiple (5 instances), the same process as described just above would be used.

Assuming the instance is booted up, and 11 terminals have been opened from the “Running the crawler on one instance” section, the first step is to stop the crawler. Use Control C in the terminal that ran the runCrawler file or kill the program using `ps -ef |grep java` and `kill [PID]`.

Open a new terminal and run the following commands to start the indexer:

```sh
javac --source-path src src/cis5550/*/*.java
jar cf indexer.jar src/cis5550/jobs/Indexer.java
java -cp src/ cis5550.flame.FlameSubmit localhost:9000 indexer.jar cis5550.jobs.Indexer
```

### Running PageRank

Assuming the instance is booted up and the indexer has finished running, open a new terminal and run the following commands to start the PageRank algorithm:

```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src src/cis5550/*/*.java
jar cf pagerank.jar classes/cis5550/jobs/PageRank.class
nohup java -cp bin:lib/kvs.jar:lib/webserver.jar:lib/flame.jar:lib/json-20140107.jar cis5550.crawling.runPageRank
```

### Running the front end

Assuming the instance is booted up and PageRank has finished running, close down the coordinators and workers (using Comand C or  `ps -ef |grep java` and `kill [PID]`) and recompile the code using the following command:

```sh
javac -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar:lib/json-20140107.jar --source-path src -d bin $(find src -name '*.java')
```

Then reboot the coordinators and workers by opening 10 new terminals and run each of the following lines in a separate terminal:
 
```sh
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Coordinator 8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8001 worker1 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8002 worker2 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8003 worker3 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar cis5550.kvs.Worker 8004 worker4 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Coordinator 9000 localhost:8000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9001 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9002 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9003 localhost:9000
nohup java -cp bin:lib/webserver.jar:lib/kvs.jar:lib/flame.jar cis5550.flame.Worker 9004 localhost:9000
```

In the index.js line `fetch('http://127.0.0.1:8003/search?query=${encodeURIComponent(searchTerm)}')` change 127.0.0.1 to the public IP address of the EC2 instance. 

Install Python, if it's not installed (`sudo apt-get install python3`), and start a Python server: 

```sh
nohup python3 -m http.server 8006 &
```

Now head over to the index.html page in a browser (http://[INSTANCE PUBLIC IP]:8006/UI/index.html) to view the completed search engine!
 
## External Resources
The only external package we used was a JAR to import JSON objects and JSON arrays, which can be downloaded from [Maven](https://mvnrepository.com/artifact/org.json/json/20140107). 
 
## Instance Types
Throughout the project, c5ad.2xlarge instances were used.
