# Python Flask Backend + JS Frontend Calculator on AWS EKS (Kubernetes) with CI/CD, Monitoring and Logging
This is a simple tutorial on how to create a Python Flask Backend + Javascript Frontend Application, Setup CI/CD with AWS Code% and AWS Elastic Kubernetes Service, High Availability, Autoscaling, Monitoring using Prometheus and Grafana and Logging with ElasticSearch, Fluentd and Kibana.

- *Step 1: Create Backend using Python Flask Rest API*
- Step 2: Create Frontend using HTML, CSS, JS

- TODO: Add Governance - AWS Organizations and AWS Account Hardening
- TODO: Add Infrastructure as Code Templates (Custom VPC with Public and Private Subnets)
- TODO: Add Testing for Infrastructure as Code
- TODO: Setup CI/CD for Infrastructure as Code

- Step 3: Install Kubernetes Tools and Launch EKS using EKCTL
- Step 4: Deploy Backend MicroService to EKS
- Step 5: Setup CI/CD for Back End Service

- TODO: Add Security - API Gateway in front of EKS Endpoint
- TODO: Add Security - Web Application Firewall in front of API Gateway
- TODO: Add Testing - Backend Static Code Analysis
- TODO: Add Testing - Backend Containers Scanning

- Step 6: Deploy Frontend to S3, CloudFront
- Step 7: Setup CI/CD for Front End Service

- TODO: Add Security - Web Application Firewall in front of CloudFront CDN
- TODO: Add Testing - Frontend Static Testing
- TODO: Add Testing - Frontend Dynamic Testing

- TODO: Add Database - NoSQL (DynamoDB)
- TODO: Add Caching - DynamoDB Accelerator (DAX)
- TODO: Add Database - RDBMS (Postgres)
- TODO: Add Caching - ElastiCache (Redis)

- TODO: Add Testing - Committed Secrets on Git Repository
- TODO: Add Security - AWS Secrets Management

- TODO: Add SSL/TLS using AWS Certificate Manager
- TODO: Add Register/Transfer Domain Name using Route 53

- Step 8: Install Helm
- Step 9: Deploy Prometheus for basic monitoring
- Step 10: Deploy Grafana to create Dashboards
- Step 11: Implement Liveness Probe Health Checks
- Step 12: Implement Readiness Probe Health Checks
- Step 13: Implement Auto Scaling
- Step 14: Logging with ElastiSearch, Fluentd, and Kibana (EFK)

- TODO: Add Authentication
- TODO: Add Instrumentation
- TODO: Add Alerting
- TODO: Add Service Discovery
- TODO: Add Service Mesh

## Prerequisites
Working Directory
```
$ cd ~
$ mkdir environment
$ cd ~/environment
```

Homebrew
```
$ brew --version
```

AWS CLI
```
$ aws --version
$ aws configure
```

Git
```
$ git --version
$ git config --global user.name "REPLACE_ME_WITH_YOUR_NAME"
$ git config --global user.email REPLACE_ME_WITH_YOUR_EMAIL@example.com
$ git config --global credential.helper '!aws codecommit credential-helper $@'
$ git config --global credential.UseHttpPath true
```

Python
```
$ python3 --version
Python 3.7.3
```

Virtualenv
```
$ virtualenv --version
16.4.3
$ echo 'venv' > .gitignore
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $
(venv) $ deactivate
```

Flask
```
(venv) $ pip install flask
(venv) $ flask --version
Flask 1.0.2
Python 3.7.3 (default, Mar 27 2019, 09:23:15) 
[Clang 10.0.1 (clang-1001.0.46.3)]
```

Docker
```
$ docker -v
Docker version 18.09.2, build 6247962
```

I am following a microservice Architecture with separate repositories and CI/CD pipelines for each microservice. The Backend Project Layout will look like this:

```
~/environment/calculator-backend
├── aws-cli/
│   ├── artifacts-bucket-policy.json
│   ├── code-build-project.json
│   └── eks-calculator-codebuild-codepipeline-iam-role.yml
├── app.py
├── buildspec.yml
├── calculator.py
├── kubernetes/
│   ├── deployment.yml
│   └── service.yml
├── requirements.txt
├── test_calculator.py
├── venv/
├── Dockerfile
├── README.md
└── .gitignore
```

Frontend Project Layout will look like this:
```
~/environment/calculator-frontend
├── aws-cli/
│   ├── artifacts-bucket-policy.json
│   ├── code-pipeline.json
│   ├── eks-calculator-codebuild-codepipeline-iam-role.yaml
│   ├── website-bucket-policy.json
│   └── website-cloudfront-distribution.json
├── base.css
├── index.html
├── querycalc.js
└── README.md
```

# ************************************************************

## Module 1: Create Backend using Python Flask REST API
- Basic calculations (add, subtract, multiply, divide)
- Advanced calculations (square root, cube root, power, factorial)
- Calculator triggered by developers via a web api

```
HTTP METHOD | URI                                                         | Action
-----------   -----------------------------------------------------------   --------------------------------------------------
POST        | http://[hostname]/add {"argument1":a, "argument2":b }       | Adds two numbers (a + b)
POST        | http://[hostname]/subtract {"argument1":a, "argument2":b }  | Subracts two numbers (a - b)
POST        | http://[hostname]/multiply {"argument1":a, "argument2":b }  | Multiplies two numbers (a * b)
POST        | http://[hostname]/divide {"argument1":a, "argument2":b }    | Divides two numbers (a / b)
POST        | http://[hostname]/sqrt {"argument1":a }                     | Gets the square root of a number (a)
POST        | http://[hostname]/cbrt {"argument1":a }                     | Gets the cube root of a number (a)
POST        | http://[hostname]/exp {"argument1":a, "argument2":b }       | Gets the the exponent of a raised to b
POST        | http://[hostname]/factorial {"argument1":a }                | Get the factorial of number 5! = 5 * 4 * 3 * 2 * 1 
```

### Step 1.1: Create a CodeCommit Repository
```
$ aws codecommit create-repository --repository-name calculator-backend
```

### Step 1.2: Clone the repository
```
$ cd ~/environment
$ git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/calculator-backend
```

### Step 1.3: Setup .gitignore
```
$ cd ~/environment/calculator-backend
$ vi .gitignore
```
```
venv
*.pyc
```

### Step 1.4: Test access to repo by adding README.md file and push to remote repository
```
$ cd ~/environment/calculator-backend
$ echo "calculator-backend" >> README.md
$ git add .
$ git commit -m "Adding README.md"
$ git push origin master
```

### Step 1.5: Navigate to working directory
```
$ cd ~/environment/calculator-backend
$ virtualenv venv
$ venv/bin/pip install flask
$ venv/bin/pip install flask-cors
```

### Step 1.6 Create Calculator Class Calculator.py
```
$ cd ~/environment/calculator-backend
$ vi calculator.py
```
```
#!/usr/bin/env python
import math

class Calculator:
    def __init__(self):
        pass

    def add(self, arg1, arg2):
        return float(arg1) + float(arg2)

    def subtract(self, arg1, arg2):
        return float(arg1) - float(arg2)

    def multiply(self, arg1, arg2):
        return float(arg1) * float(arg2)

    def divide(self, arg1, arg2):
        return float(arg1) / (arg2)

    def sqrt(self, arg1):
        return math.sqrt(float(arg1))

    def cbrt(self, arg1):
        return round(float(arg1)**(1.0/3))

    def exp(self, arg1, arg2):
        return float(arg1) ** float(arg2)

    def factorial(self, arg1):
       if arg1 == 1:
          return arg1
       else:
          return float(arg1)*self.factorial(float(arg1)-1)
```

### Step 1.7: Add app.py
```
$ cd ~/environment/calculator-backend
$ vi app.py
```
```
#!/usr/bin/env python
from flask import (Flask, jsonify, request, abort, render_template, logging)
from flask_cors import CORS
from calculator import Calculator

app = Flask(__name__)
CORS(app)

@app.route('/')
def index_page():
    return "This is a RESTful Calculator App built with Python Flask! - testing my CI/CD Pipeline"

@app.route('/add', methods=['POST'])
def add_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        arg2 = request.json['argument2']
        calculator = Calculator()
        answer = calculator.add(arg1, arg2)
        app.logger.info('{ "operation": "add", "arg1": "%s", "arg2": "%s", "answer": "%s" }', arg1, arg2, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/subtract', methods=['POST'])
def subtract_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        arg2 = request.json['argument2']
        calculator = Calculator()
        answer = calculator.subtract(arg1, arg2)
        app.logger.info('{ "operation": "subtract", "arg1": "%s", "arg2": "%s", "answer": "%s" }', arg1, arg2, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/multiply', methods=['POST'])
def multiply_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        arg2 = request.json['argument2']
        calculator = Calculator()
        answer = calculator.multiply(arg1, arg2)
        app.logger.info('{ "operation": "multiply", "arg1": "%s", "arg2": "%s", "answer": "%s" }', arg1, arg2, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/divide', methods=['POST'])
def divide_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        arg2 = request.json['argument2']
        calculator = Calculator()
        answer = calculator.divide(arg1, arg2)
        app.logger.info('{ "operation": "divide", "arg1": "%s", "arg2": "%s", "answer": "%s" }', arg1, arg2, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)
    except ZeroDivisionError:
        abort(400)

@app.route('/sqrt', methods=['POST'])
def sqrt_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        calculator = Calculator()
        answer = calculator.sqrt(arg1)
        app.logger.info('{ "operation": "sqrt", "arg1": "%s", "arg2": "none", "answer": "%s" }', arg1, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/cbrt', methods=['POST'])
def cbrt_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        calculator = Calculator()        
        answer = calculator.cbrt(arg1)
        app.logger.info('{ "operation": "cbrt", "arg1": "%s", "arg2": "none", "answer": "%s" }', arg1, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/exp', methods=['POST'])
def exponent_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        arg2 = request.json['argument2']
        calculator = Calculator()        
        answer = calculator.exp(arg1, arg2)
        app.logger.info('{ "operation": "exp", "arg1": "%s", "arg2": "%s", "answer": "%s" }', arg1, arg2, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

@app.route('/factorial', methods=['POST'])
def factorial_args():
    if not request.json:
        abort(400)
    try:
        arg1 = request.json['argument1']
        calculator = Calculator()          
        answer = calculator.factorial(arg1)
        app.logger.info('{ "operation": "factorial", "arg1": "%s", "arg2": "none", "answer": "%s" }', arg1, answer)
        return (jsonify({'answer':answer}), 200)
    except KeyError:
        abort(400)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

### Step 1.8: Create the requirements.txt file
```
$ cd ~/environment/calculator-backend
$ vi requirements.txt
```
```
flask
flask_restful
flask_cors
```

### Step 1.9: Run Locally and Test
```
$ cd ~/environment/calculator-backend
$ chmod a+x app.py
$ ./app.py
$ curl http://localhost:5000
```

### Step 1.10: Backend Unit Tests
```
$ cd ~/environment/calculator-backend
$ vi test_calculator.py
```
```
#!/usr/bin/env python
import unittest
from calculator import Calculator

class CalculatorTest(unittest.TestCase):
    calculator = Calculator()
    
    def test_add(self):
        self.assertEqual(4, self.calculator.add(2,2))

    def test_subtract(self):
        self.assertEqual(2, self.calculator.subtract(3,1))
        self.assertEqual(-2, self.calculator.subtract(1,3))

    def test_multiply(self):
        self.assertEqual(12, self.calculator.multiply(3,4))
        self.assertEqual(13.5, self.calculator.multiply(3,4.5))

    def test_divide(self):
        self.assertEqual(3, self.calculator.divide(9,3))
        with self.assertRaises(ZeroDivisionError):
            self.calculator.divide(3,0) 
    
    def test_sqrt(self):
        self.assertEqual(4, self.calculator.sqrt(16))

    def test_cbrt(self):
        self.assertEqual(4, self.calculator.cbrt(64))

    def test_exp(self):
        self.assertEqual(32, self.calculator.exp(2,5))

    def test_factorial(self):
        self.assertEqual(120, self.calculator.factorial(5))

if __name__ == "__main__":
    unittest.main()
```

### Step 1.11: Run Unit Tests
```
$ chmod a+x test_calculator.py
$ ./test_calculator.py -v
```

### Step 1.12 Save changes to remote git repository
```
$ git add .
$ git commit -m "Initial"
$ git push origin master
```

### Step 1.13: Create the Dockerfile
```
$ cd ~/environment/calculator-backend
$ vi Dockerfile
```
```
# Set base image to python
FROM python:2.7

# Copy source file and python req's
COPY . /app
WORKDIR /app

# Install requirements
RUN pip install -r requirements.txt

# Set image's main command and run the command within the container
ENTRYPOINT ["python"]
CMD ["app.py"]
```

### Step 1.14: Build, Tag and Run the Docker Image locally
Replace:
- AccountId: 707538076348
- Region: us-east-1

```
$ docker build -t calculator-backend .
$ docker tag calculator-backend:latest 707538076348.dkr.ecr.us-east-1.amazonaws.com/calculator-backend:latest
$ docker run -d -p 5000:5000 calculator-backend:latest
```

### Step 1.15: Test Math Operations
- Test Add
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":2, "argument2":1 }' http://localhost:5000/add
{
  "answer": 3
}
```

- Test Subtract
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":4, "argument2":3 }' http://localhost:5000/subtract
{
  "answer": 1
}
```

- Test Multiply
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":2, "argument2":3 }' http://localhost:5000/multiply
{
  "answer": 6
}
```

- Test Divide
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":12, "argument2":4 }' http://localhost:5000/divide
{
  "answer": 3
}
```

- Test Square Root
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":9 }' http://localhost:5000/sqrt
{
  "answer": 3
}
```

- Test Cube Root
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":64 }' http://localhost:5000/cbrt
{
  "answer": 4
}
```

- Test Power
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":2, "argument2":3 }' http://localhost:5000/exp
{
  "answer": 8
}
```

- Test Factorial
```
$ curl -i -H "Content-Type: application/json" -X POST -d '{"argument1":5 }' http://localhost:5000/factorial
{
  "answer": 120
}
```

### Step 1.16: Create the ECR Repository
```
$ aws ecr create-repository --repository-name calculator-backend
```

### Step 1.17: Run login command to retrieve credentials for our Docker client and then automatically execute it (include the full command including the $ below).
```
$ $(aws ecr get-login --no-include-email)
```

### Step 1.18: Push our Docker Image
```
$ docker push 707538076348.dkr.ecr.us-east-1.amazonaws.com/calculator-backend:latest
```

### Step 1.19: Validate Image has been pushed
```
$ aws ecr describe-images --repository-name calculator-backend
```

### (Optional) Clean up
```
$ aws ecr delete-repository --repository-name calculator-backend --force
$ aws codecommit delete-repository --repository-name calculator-backend
$ rm -rf ~/environment/calculator-backend
```

# ************************************************************

## Module 2: Create Frontend using HTML, CSS, JS
- Calculator triggered by end users through a web page
- TODO: Use Bootstrap for UI

### Step 2.1: Create a CodeCommit Repository
```
$ aws codecommit create-repository --repository-name calculator-frontend
```

### Step 2.2: Clone the repository
```
$ cd ~/environment
$ git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/calculator-frontend
```

### Step 2.3: Test access to repo by adding README.md file and push to remote repository
```
$ cd ~/environment/calculator-frontend
$ echo "calculator-frontend" >> README.md
$ git add .
$ git commit -m "Adding README.md"
$ git push origin master
```

### Step 2.4: Navigate to working directory
```
$ cd ~/environment/calculator-frontend
```

### Step 2.5: Create index.html file
```
$ vi index.html
```
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Simple RESTful Calculator - Web Client</title>
    </head>
    <body>
        <div class="outer-container">
            <h1>Simple RESTful Calculator</h1>
            <form id="calcform" method="POST">
            <div class="arguments">
                Argument 1: <input type="text" id="argument1" value = ""/><br/>
                Argument 2: <input type="text" id="argument2" value = ""/><br/>
            </div>
            <div class="buttons">
                <div class="button-row">
                    <input class= "func-btn" type="button" id="add" value="Add"/>
                    <input class= "func-btn" type="button" id="subtract" value="Subtract"/>
                </div>
                <div class="button-row">
                    <input class= "func-btn" type="button" id="multiply" value="Multiply"/>
                    <input class= "func-btn" type="button" id="divide" value="Divide"/>
                </div>
                <div class="button-row">
                    <input class= "func-btn" type="button" id="sqrt" value="SquareRoot"/>
                    <input class= "func-btn" type="button" id="cbrt" value="CubeRoot"/>
                </div>		
                <div class="button-row">
                    <input class= "func-btn" type="button" id="exp" value="Exponent"/>
                    <input class= "func-btn" type="button" id="factorial" value="Factorial"/>		    
                </div>
            </div>
            </form>
        </div>

        <link rel="stylesheet" type="text/css" href="./base.css">
        <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>
        <script src="./querycalc.js"></script>
        <script type="text/javascript">
            // once the DOM is ready start the calculator listener function
            $( document ).ready( calcListener ); 
        </script>

    </body>
</html>
```

### Step 2.6: Create base.css file
```
$ vi base.css
```
```
body {
    margin: 0;
    padding: 0;
    background-color: white;
}

.outer-container {
    width: 500px;
    height: 500px;
    margin-left: auto;
    margin-right: auto;
    overflow: auto;
    text-align: center;
    background-color: lightblue;
}

.arguments {
    margin: auto;
    padding: 5px;
}

.buttons {
    padding: 5px;
}

.button-row {
    height: 50px;
}

.func-btn {
	-webkit-appearance: none;
    width: 50%;
    height: 50px;
    float: left;
    padding: 5px;
}
```

### Step 2.7: Create querycalc.js file
```
$ vi querycalc.js
```
```
function calcListener ( jQuery ) {
    console.log( "READY!" );

    $( "#add" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'add');
        e.preventDefault();
    });

    $( "#subtract" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'subtract');
        e.preventDefault();
    });

    $( "#multiply" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'multiply');
        e.preventDefault();
    });

    $( "#divide" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'divide');
        e.preventDefault();
    });
    
    $( "#sqrt" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'sqrt');
        e.preventDefault();
    });
    
    $( "#cbrt" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'cbrt');
        e.preventDefault();
    });       
    
    $( "#exp" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        var arg2 = $( "#argument2" ).val();
        doMath (arg1, arg2, 'exp');
        e.preventDefault();
    });
    
    $( "#factorial" ).click( function ( e ) {
        var arg1 = $( "#argument1" ).val();
        doMath (arg1, 1, 'factorial');
        e.preventDefault();
    })    
    
    function doMath( arg1, arg2, resource ) {
        var textStatus, jqXHR, errorThrown = '';

        console.log( "Calling " + resource + " on " + arg1 + " and " + arg2 );

        
        try {
            arg1 = Number( arg1 );
            arg2 = Number( arg2 );
	    
            //TODO handle non-numeric inputs
            if ( isNaN(arg1) || isNaN(arg2) ) throw "NaN";

            // Flask requires a JSON string and the following content-type
            $.ajaxSetup({
                contentType: "application/json"
            });

            // Makes an ajax call to url "resource" supplying arg1 and arg2
            $.ajax({
                type: "POST",
                url: "http://localhost:5000/" + resource,
                data: JSON.stringify({ argument1: arg1, argument2: arg2 }),
                dataType: "json",
                success: function ( data ) {
                    var answer = String(data[ 'answer' ]);
                    console.log( "We got an answer! " + answer );

                    // Put the answer in argument1 and blank out argument2
                    $( "#argument1" ).val( answer );
                    $( "#argument2" ).val( '' );
                },
                error: function ( textStatus, jqXHR, errorThrown ) {
                    console.log("Something has gone wrong:" + errorThrown);
                    $( "#argument1" ).val( '' );
                    $( "#argument2" ).val( '' );
                }
            });
        }

        catch( err ) {
            console.log("Error: " + err);
            $( "#argument1" ).val( '' );
            $( "#argument2" ).val( '' );
        }
    }
}
```

### Step 2.8: Test Locally

### Step 2.9: Save changes to remote git repository
```
$ git add .
$ git commit -m "Initial"
$ git push origin master
```

### (Optional) Clean up
```
$ aws codecommit delete-repository --repository-name calculator-frontend
$ Manually delete SSH Key
$ rm ~/environment/calculator-frontend/querycalc.js
$ rm ~/environment/calculator-frontend/base.css
$ rm ~/environment/calculator-frontend/index.html
$ rm -rf ~/environment/calculator-frontend
```

# ************************************************************

## Module 3: Install Kubernetes Tools and Launch EKS using EKCTL

### Step 3.1: Create the default ~/.kube directory for storing kubectl configuration
```
$ mkdir -p ~/.kube
```

### Step 3.2: Install kubectl on MAC
```
$ sudo curl -o kubectl "https://amazon-eks.s3-us-west-2.amazonaws.com/1.11.5/2018-12-06/bin/darwin/amd64/kubectl"
$ sudo chmod +x ./kubectl
$ mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
$ echo 'export PATH=$HOME/bin:$PATH' >> ~/.bash_profile
$ kubectl version --short --client
```

### Step 3.3: Install IAM Authenticator
```
$ brew install aws-iam-authenticator
$ aws-iam-authenticator help
```

### Step 3.4: Install JQ and envsubst
```
$ brew install jq
$ brew install gettext
$ brew link --force gettext
```

### Step 3.5: Verify the binaries are in the path and executable
```
$ for command in kubectl aws-iam-authenticator jq envsubst
  do
    which $command &>/dev/null && echo "$command in path" || echo "$command NOT FOUND"
  done
```

### Step 3.6: Generate an SSH Key for the Worker Nodes and upload the public key to your EC2 region
```
$ ssh-keygen
$ aws ec2 import-key-pair --key-name "eksworkernodes" --public-key-material file://~/.ssh/id_rsa.pub
```

### Step 3.7: Download the eksctl binaries
```
$ brew install weaveworks/tap/eksctl
$ eksctl version
```

### Step 3.8: Create an EKS Cluster (This will take ~15 minutes) and test cluster
```
$ eksctl create cluster \
--name=calculator-eksctl \
--nodes=2 \
--node-ami=auto \
--node-type=t2.medium \ 
--region=${AWS_REGION}
$ kubectl get nodes
```

```
[ℹ]  using region us-east-1
[ℹ]  setting availability zones to [us-east-1d us-east-1a]
[ℹ]  subnets for us-east-1d - public:192.168.0.0/19 private:192.168.64.0/19
[ℹ]  subnets for us-east-1a - public:192.168.32.0/19 private:192.168.96.0/19
[ℹ]  nodegroup "ng-2e8e1187" will use "ami-0abcb9f9190e867ab" [AmazonLinux2/1.12]
[ℹ]  creating EKS cluster "calculator-eksctl" in "us-east-1" region
[ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial nodegroup
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --name=calculator-eksctl'
[ℹ]  2 sequential tasks: { create cluster control plane "calculator-eksctl", create nodegroup "ng-2e8e1187" }
[ℹ]  building cluster stack "eksctl-calculator-eksctl-cluster"
[ℹ]  deploying stack "eksctl-calculator-eksctl-cluster"
[ℹ]  buildings nodegroup stack "eksctl-calculator-eksctl-nodegroup-ng-2e8e1187"
[ℹ]  --nodes-min=2 was set automatically for nodegroup ng-2e8e1187
[ℹ]  --nodes-max=2 was set automatically for nodegroup ng-2e8e1187
[ℹ]  deploying stack "eksctl-calculator-eksctl-nodegroup-ng-2e8e1187"
[✔]  all EKS cluster resource for "calculator-eksctl" had been created
[✔]  saved kubeconfig as "/Users/jrdalino/.kube/config"
[ℹ]  adding role "arn:aws:iam::707538076348:role/eksctl-calculator-eksctl-nodegrou-NodeInstanceRole-1IGO2PUALGGAC" to auth ConfigMap
[ℹ]  nodegroup "ng-2e8e1187" has 1 node(s)
[ℹ]  node "ip-192-168-24-30.ec2.internal" is not ready
[ℹ]  waiting for at least 2 node(s) to become ready in "ng-2e8e1187"
[ℹ]  nodegroup "ng-2e8e1187" has 2 node(s)
[ℹ]  node "ip-192-168-24-30.ec2.internal" is ready
[ℹ]  node "ip-192-168-35-7.ec2.internal" is ready
[ℹ]  kubectl command should work with "/Users/jrdalino/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "calculator-eksctl" in "us-east-1" region is ready
```

### Step 3.9: Export Worker Role name
```
$ INSTANCE_PROFILE_NAME=$(aws iam list-instance-profiles | jq -r '.InstanceProfiles[].InstanceProfileName' | grep nodegroup)
$ ROLE_NAME=$(aws iam get-instance-profile --instance-profile-name $INSTANCE_PROFILE_NAME | jq -r '.InstanceProfile.Roles[] | .RoleName')
$ echo "export ROLE_NAME=${ROLE_NAME}" >> ~/.bash_profile
```

### (Optional) Clean up
```
$ eksctl delete cluster --name=calculator-eksctl
$ Manually delete Worker Nodes SSH Key
```

# ************************************************************

## Module 4: Deploy Backend MicroService to EKS
- containerized with Docker and Kubernetes for the orchestration

### Step 4.1: Create our deployment.yaml file
```
$ cd ~/environment/calculator-backend
$ mkdir kubernetes
$ vi ~/environment/calculator-backend/kubernetes/deployment.yaml
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: calculator-backend
  labels:
    app: calculator-backend
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: calculator-backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: calculator-backend
    spec:
      containers:
      - image: 707538076348.dkr.ecr.us-east-1.amazonaws.com/calculator-backend:latest
        imagePullPolicy: Always
        name: calculator-backend
        ports:
        - containerPort: 5000
          protocol: TCP
```

### Step 4.2: Create our service.yaml file
```
$ vi ~/environment/calculator-backend/kubernetes/service.yaml
```

```
apiVersion: v1
kind: Service
metadata:
  name: calculator-backend
spec:
  selector:
    app: calculator-backend
  type: LoadBalancer
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 5000
```

### Step 4.3: Ensure ELB service Role exists
```
$ aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing" || aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"
```

### Step 4.4: Deploy our Backend REST API and watch progress
```
$ cd ~/environment/calculator-backend/kubernetes
$ kubectl apply -f deployment.yaml
$ kubectl apply -f service.yaml
$ kubectl get deployment calculator-backend
```

### Step 4.5: Scale the Backend Service
```
$ kubectl get deployments
$ kubectl scale deployment calculator-backend --replicas=1
$ kubectl get deployments
```

### Step 4.6: Find the Service Address
```
$ kubectl get service calculator-backend -o wide
```

### (Optional) Clean up
```
$ cd ~/environment/calculator-backend/kubernetes
$ kubectl delete -f service.yaml
$ kubectl delete -f deployment.yaml
```

# ************************************************************

## Module 5: Deploy Frontend

### Step 5.1: Replace http://localhost:5000 url with ELB Endpoint Ex. http://a529520be6d7811e98ef812788873e53-1902855455.us-east-1.elb.amazonaws.com/
```
$ vi ~/environment/calculator-frontend/querycalc.js
```

### Step 5.2: Create an S3 Bucket for Storing Content
```
$ aws s3 mb s3://jrdalino-calculator-frontend
```

### Step 5.3: Create a CloudFront Access Identity
```
$ aws cloudfront create-cloud-front-origin-access-identity \
--cloud-front-origin-access-identity-config CallerReference=Calculator,Comment=Calculator
```

### Step 5.4: Create the S3 Bucket Policy Input File
```
$ mkdir aws-cli
$ vi ~/environment/calculator-frontend/aws-cli/website-bucket-policy.json
```
```
{
    "Version": "2008-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity EXZ8BOEUVCLQY"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::jrdalino-calculator-frontend/*"
        }
    ]
}
```

### Step 5.5: Add a public bucket policy to allow CloudFront
```
$ aws s3api put-bucket-policy \
--bucket jrdalino-calculator-frontend \
--policy file://~/environment/calculator-frontend/aws-cli/website-bucket-policy.json
```

### Step 5.6: Publish the Website Content to S3
```
$ cd ~/environment/calculator-frontend
$ aws s3 cp index.html s3://jrdalino-calculator-frontend/index.html
$ aws s3 cp base.css s3://jrdalino-calculator-frontend/base.css
$ aws s3 cp querycalc.js s3://jrdalino-calculator-frontend/querycalc.js
```

### Step 5.7: Create the CloudFront Distribution input file
```
$ cd ~/environment/calculator-frontend
$ mkdir aws-cli
$ vi ~/environment/calculator-frontend/aws-cli/website-cloudfront-distribution.json
```

```
{
  "CallerReference": "Calculator",
  "Aliases": {
    "Quantity": 0
  },
  "DefaultRootObject": "index.html",
  "Origins": {
    "Quantity": 1,
    "Items": [
      {
        "Id": "Calculator",
        "DomainName": "jrdalino-calculator-frontend.s3.amazonaws.com",
        "S3OriginConfig": {
          "OriginAccessIdentity": "origin-access-identity/cloudfront/EXZ8BOEUVCLQY"
        }
      }
    ]
  },
  "DefaultCacheBehavior": {
    "TargetOriginId": "Calculator",
    "ForwardedValues": {
      "QueryString": true,
      "Cookies": {
        "Forward": "none"
      }
    },
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "ViewerProtocolPolicy": "allow-all",
    "MinTTL": 0,
    "MaxTTL": 0,
    "DefaultTTL": 0
  },
  "CacheBehaviors": {
    "Quantity": 0
  },
  "Comment": "",
  "Logging": {
    "Enabled": false,
    "IncludeCookies": true,
    "Bucket": "",
    "Prefix": ""
  },
  "PriceClass": "PriceClass_All",
  "Enabled": true
}
```

### Step 5.8: Create CloudFront Distribution
```
$ aws cloudfront create-distribution \
--distribution-config file://~/environment/calculator-frontend/aws-cli/website-cloudfront-distribution.json
```

### Step 5.9: Check Status of CloudFront Distribution
```
$ aws cloudfront list-distributions
```

### Step 5.10 Enable CORS on S3 and CloudFront
- Cloudfront: https://aws.amazon.com/premiumsupport/knowledge-center/no-access-control-allow-origin-error/

- S3: https://docs.aws.amazon.com/AmazonS3/latest/user-guide/add-cors-configuration.html
```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

### Step 5.11: Test functionality of Calculator Frontend + Backend
```
$ curl d5ny4mdta1kxt.cloudfront.net
```

### (Optional) Clean up
```
$ aws s3 rm s3://jrdalino-calculator-frontend --recursive
$ aws s3 rb s3://jrdalino-calculator-frontend --force
$ rm ~/environment/calculator-frontend/aws-cli/website-bucket-policy.json
$ disable cloudfront distribution
$ delete cloudfront distribution
$ aws cloudfront delete-cloud-front-origin-access-identity --id EXZ8BOEUVCLQY
$ rm ~/environment/calculator-frontend/aws-cli/website-cloudfront-distribution.json
```

# ************************************************************

## Module 6: Setup CI/CD for Back End Service
- proper CI/CD processes to put in place

### Step 6.1: Create Codebuild and Codepipeline Role (eks-calculator-codebuild-codepipeline-iam-role)
```
$ cd ~/environment/calculator-backend
$ mkdir aws-cli
$ vi ~/environment/calculator-backend/aws-cli/eks-calculator-codebuild-codepipeline-iam-role.yaml
```
```
---
AWSTemplateFormatVersion: '2010-09-09'
Resources:

  # An IAM role that allows the AWS CodeBuild service to perform the actions
  # required to complete a build of our source code retrieved from CodeCommit,
  # and push the created image to ECR.

  CalculatorServiceCodeBuildServiceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: CalculatorServiceCodeBuildServiceRole
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          Effect: Allow
          Principal:
            Service: codebuild.amazonaws.com
          Action: sts:AssumeRole
      Policies:
      - PolicyName: "CalculatorService-CodeBuildServicePolicy"
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
          - Effect: "Allow"
            Action:
            - "codecommit:ListBranches"
            - "codecommit:ListRepositories"
            - "codecommit:BatchGetRepositories"
            - "codecommit:Get*"
            - "codecommit:GitPull"
            Resource:
            - Fn::Sub: arn:aws:codecommit:${AWS::Region}:${AWS::AccountId}:CalculatorServiceRepository
          - Effect: "Allow"
            Action:
            - "logs:CreateLogGroup"
            - "logs:CreateLogStream"
            - "logs:PutLogEvents"
            Resource: "*"
          - Effect: "Allow"
            Action:
            - "s3:PutObject"
            - "s3:GetObject"
            - "s3:GetObjectVersion"
            - "s3:ListBucket"
            Resource: "*"
          - Effect: "Allow"
            Action:
            - "ecr:GetAuthorizationToken"
            - "ecr:InitiateLayerUpload"
            - "ecr:UploadLayerPart"
            - "ecr:CompleteLayerUpload"
            - "ecr:BatchCheckLayerAvailability"
            - "ecr:PutImage"
            Resource: "*"

  # An IAM role that allows the AWS CodePipeline service to perform it's
  # necessary actions. 

  CalculatorServiceCodePipelineServiceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: CalculatorServiceCodePipelineServiceRole
      AssumeRolePolicyDocument:
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - codepipeline.amazonaws.com
          Action:
          - sts:AssumeRole
      Path: "/"
      Policies:
      - PolicyName: CalculatorService-codepipeline-service-policy
        PolicyDocument:
          Statement:
          - Action:
            - codecommit:GetBranch
            - codecommit:GetCommit
            - codecommit:UploadArchive
            - codecommit:GetUploadArchiveStatus
            - codecommit:CancelUploadArchive
            Resource: "*"
            Effect: Allow
          - Action:
            - s3:GetObject
            - s3:GetObjectVersion
            - s3:GetBucketVersioning
            Resource: "*"
            Effect: Allow
          - Action:
            - s3:PutObject
            Resource:
            - arn:aws:s3:::*
            Effect: Allow
          - Action:
            - elasticloadbalancing:*
            - autoscaling:*
            - cloudwatch:*
            - ecs:*
            - eks:*
            - codebuild:*
            - codepipeline:*
            - codedeploy:*
            - iam:ListRoles	    
            - iam:PassRole
            - lambda:*
            - sns:*
            Resource: "*"
            Effect: Allow
          Version: "2012-10-17"
```

```
$ aws cloudformation create-stack \
--stack-name eks-calculator-codebuild-codepipeline-iam-role \
--capabilities CAPABILITY_NAMED_IAM \
--template-body file://~/environment/calculator-backend/aws-cli/eks-calculator-codebuild-codepipeline-iam-role.yaml
```

### Step 6.2: Create an S3 Bucket for Pipeline Artifacts
```
$ aws s3 mb s3://jrdalino-calculator-backend-artifacts
```

### Step 6.3: Modify S3 Bucket Policy
```
$ vi ~/environment/calculator-backend/aws-cli/artifacts-bucket-policy.json
```

```
{
    "Statement": [
      {
        "Sid": "WhitelistedGet",
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "arn:aws:iam::707538076348:role/CalculatorServiceCodeBuildServiceRole",
            "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole"
          ]
        },
        "Action": [
          "s3:GetObject",
          "s3:GetObjectVersion",
          "s3:GetBucketVersioning"
        ],
        "Resource": [
          "arn:aws:s3:::jrdalino-calculator-backend-artifacts/*",
          "arn:aws:s3:::jrdalino-calculator-backend-artifacts"
        ]
      },
      {
        "Sid": "WhitelistedPut",
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "arn:aws:iam::707538076348:role/CalculatorServiceCodeBuildServiceRole",
            "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole"
          ]
        },
        "Action": "s3:PutObject",
        "Resource": [
          "arn:aws:s3:::jrdalino-calculator-backend-artifacts/*",
          "arn:aws:s3:::jrdalino-calculator-backend-artifacts"
        ]
      }
    ]
}
```

### Step 6.4: Grant S3 Bucket access to your CI/CD Pipeline
```
$ aws s3api put-bucket-policy \
--bucket jrdalino-calculator-backend-artifacts \
--policy file://~/environment/calculator-backend/aws-cli/artifacts-bucket-policy.json
```

### Step 6.5: View/Modify Buildspec file
```
$ cd ~/environment/calculator-backend
$ vi ~/environment/calculator-backend/buildspec.yml
```

```
version: 0.2
phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - $(aws ecr get-login --no-include-email --region $AWS_DEFAULT_REGION)
      - TAG="$(date +%s)"
      - REPOSITORY_URI="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME"
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - docker build -t $REPOSITORY_URI:$TAG .
  post_build:
    commands:
      - echo Uploading the Docker image...
      - docker push $REPOSITORY_URI:$TAG
      - printf '[{"name":"calculator-backend","imageUri":"%s"}]' $REPOSITORY_URI:$TAG > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```

### Step 6.6: View/Modify CodeBuild Project Input File
```
$ vi ~/environment/calculator-backend/aws-cli/code-build-project.json
```

```
{
  "name": "CalculatorBackendServiceCodeBuildProject",
  "artifacts": {
    "type": "no_artifacts"
  },
  "environment": {
    "computeType": "BUILD_GENERAL1_SMALL",
    "image": "aws/codebuild/python:3.5.2",
    "privilegedMode": true,
    "environmentVariables": [
      {
        "name": "AWS_ACCOUNT_ID",
        "value": "707538076348"
      },
      {
        "name": "AWS_DEFAULT_REGION",
        "value": "us-east-1"
      },      
      {
        "name": "IMAGE_REPO_NAME",
        "value": "calculator-backend"
      }
    ],
    "type": "LINUX_CONTAINER"
  },
  "serviceRole": "arn:aws:iam::707538076348:role/CalculatorServiceCodeBuildServiceRole",
  "source": {
    "type": "CODECOMMIT",
    "location": "https://git-codecommit.us-east-1.amazonaws.com/v1/repos/calculator-backend"
  }
}
```

### Step 6.7: Create the CodeBuild Project
```
$ aws codebuild create-project \
--cli-input-json file://~/environment/calculator-backend/aws-cli/code-build-project.json
```

### Step 6.8: Setup Lambda for deployment
```
$ cd ~/environment/
$ git clone https://github.com/BranLiang/lambda-eks
$ cd lambda-eks
$ sed -i -e "s#\$EKS_CA#$(aws eks describe-cluster --name calculator-eksctl --query cluster.certificateAuthority.data --output text)#g" ./config
$ sed -i -e "s#\$EKS_CLUSTER_HOST#$(aws eks describe-cluster --name calculator-eksctl --query cluster.endpoint --output text)#g" ./config
$ sed -i -e "s#\$EKS_CLUSTER_NAME#calculator-eksctl#g" ./config
$ sed -i -e "s#\$EKS_CLUSTER_USER_NAME#lambda#g" ./config
```

### Step 6.9: Then run the following command replacing secret name to update your token
```
$ kubectl get secrets
$ sed -i -e "s#\$TOKEN#$(kubectl get secret $SECRET_NAME -o json | jq -r '.data["token"]' | base64 -D)#g" ./config
```

### Step 6.10: Build, package and deploy the Lambda Kube Client Function
```
$ npm install
$ zip -r ../lambda-package_v1.zip .
$ cd ..
$ aws lambda create-function \
--function-name LambdaKubeClient \
--runtime nodejs8.10 \
--role arn:aws:iam::707538076348:role/lambda_admin_execution \
--handler index.handler \
--zip-file fileb://lambda-package_v1.zip \
--timeout 10 \
--memory-size 128
```

### Step 6.11: Provide admin access to default service account
```
$ kubectl create clusterrolebinding default-admin --clusterrole cluster-admin --serviceaccount=default:default
```

### Step 6.12: Modify CodePipeline Input File
```
$ vi ~/environment/calculator-backend/aws-cli/code-pipeline.json
```

```
{
  "pipeline": {
      "name": "CalculatorBackendServiceCICDPipeline",
      "roleArn": "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole",
      "stages": [
        {
          "name": "Source",
          "actions": [
            {
              "inputArtifacts": [
    
              ],
              "name": "Source",
              "actionTypeId": {
                "category": "Source",
                "owner": "AWS",
                "version": "1",
                "provider": "CodeCommit"
              },
              "outputArtifacts": [
                {
                  "name": "CalculatorBackendService-SourceArtifact"
                }
              ],
              "configuration": {
                "BranchName": "master",
                "RepositoryName": "calculator-backend"
              },
              "runOrder": 1
            }
          ]
        },
        {
          "name": "Build",
          "actions": [
            {
              "name": "Build",
              "actionTypeId": {
                "category": "Build",
                "owner": "AWS",
                "version": "1",
                "provider": "CodeBuild"
              },
              "outputArtifacts": [
                {
                  "name": "CalculatorBackendService-BuildArtifact"
                }
              ],
              "inputArtifacts": [
                {
                  "name": "CalculatorBackendService-SourceArtifact"
                }
              ],
              "configuration": {
                "ProjectName": "CalculatorBackendServiceCodeBuildProject"
              },
              "runOrder": 1
            }
          ]
        }
      ],
      "artifactStore": {
        "type": "S3",
        "location": "jrdalino-calculator-backend-artifacts"
      }
  }
}
```

### Step 6.13: Create a pipeline in CodePipeline
```
$ aws codepipeline create-pipeline \
--cli-input-json file://~/environment/calculator-backend/aws-cli/code-pipeline.json
```

### Step 6.14: Manually modify pipeline Codepipeline to add Deployment stage using created Lambda function.
- Click Edit CodePipeline
- Add a new stage after Build Stage
- Enter stage name as Deploy and save.
- Add an Action group within the stage.
- Action name: LambdaClient
- Action provider: AWS Lambda
- Region: US East - (N. Virginia)
- Input artifact: CalculatorBackendService-BuildArtifact
- Function name: LambdaKubeClient
- User parameter: calculator-backend
- Click Save

### Step 6.15: Make a small code change, push and validate changes

### (Optional) Clean up
```
$ aws codepipeline delete-pipeline --name CalculatorBackendServiceCICDPipeline
$ rm ~/environment/calculator-backend/aws-cli/code-pipeline.json
$ aws lambda delete-function --function-name LambdaKubeClient
$ rm ~/environment/lambda-package_v1.zip
$ aws codebuild delete-project --name CalculatorBackendServiceCodeBuildProject
$ Manually delete codebuild history
$ aws s3api delete-bucket-policy --bucket jrdalino-calculator-backend-artifacts
$ rm ~/environment/calculator-backend/aws-cli/artifacts-bucket-policy.json
$ aws s3 rm s3://jrdalino-calculator-backend-artifacts --recursive
$ aws s3 rb s3://jrdalino-calculator-backend-artifacts --force
$ aws lambda delete-function --function-name LogsToElasticsearch_kubernetes-logs
$ aws logs delete-log-group --log-group-name /aws/lambda/LambdaKubeClient
$ aws logs delete-log-group --log-group-name /aws/codebuild/CalculatorBackendServiceCodeBuildProject
```

# ************************************************************

## Module 7: Setup CI/CD for Front End

### Step 7.1: Create an S3 Bucket for Pipeline Artifacts
```
$ aws s3 mb s3://jrdalino-calculator-frontend-artifacts
```

### Step 7.2: Check Codepipeline Roles exists

### Step 7.3: Create S3 Bucket Policy File
```
$ cd ~/environment/calculator-frontend
$ mkdir aws-cli
$ vi ~/environment/calculator-frontend/aws-cli/artifacts-bucket-policy.json
```

```
{
    "Statement": [
      {
        "Sid": "WhitelistedGet",
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole"
          ]
        },
        "Action": [
          "s3:GetObject",
          "s3:GetObjectVersion",
          "s3:GetBucketVersioning"
        ],
        "Resource": [
          "arn:aws:s3:::jrdalino-calculator-frontend-artifacts/*",
          "arn:aws:s3:::jrdalino-calculator-frontend-artifacts"
        ]
      },
      {
        "Sid": "WhitelistedPut",
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole"
          ]
        },
        "Action": "s3:PutObject",
        "Resource": [
          "arn:aws:s3:::jrdalino-calculator-frontend-artifacts/*",
          "arn:aws:s3:::jrdalino-calculator-frontend-artifacts"
        ]
      }
    ]
}
```

### Step 7.4: Grant S3 Bucket access to your CI/CD Pipeline
```
$ aws s3api put-bucket-policy \
--bucket jrdalino-calculator-frontend-artifacts \
--policy file://~/environment/calculator-frontend/aws-cli/artifacts-bucket-policy.json
```

### Step 7.5: Create CodePipeline Input File
```
$ vi ~/environment/calculator-frontend/aws-cli/code-pipeline.json
```

```
{
  "pipeline": {
      "name": "CalculatorFrontendServiceCICDPipeline",
      "roleArn": "arn:aws:iam::707538076348:role/CalculatorServiceCodePipelineServiceRole",
      "stages": [
        {
          "name": "Source",
          "actions": [
            {
              "inputArtifacts": [
    
              ],
              "name": "Source",
              "actionTypeId": {
                "category": "Source",
                "owner": "AWS",
                "version": "1",
                "provider": "CodeCommit"
              },
              "outputArtifacts": [
                {
                  "name": "CalculatorFrontendService-SourceArtifact"
                }
              ],
              "configuration": {
                "BranchName": "master",
                "RepositoryName": "calculator-frontend"
              },
              "runOrder": 1
            }
          ]
        },
        {
          "name": "Deploy",
          "actions": [
            {
              "name": "Deploy",
              "actionTypeId": {
                "category": "Deploy",
                "owner": "AWS",
                "version": "1",
                "provider": "S3"
              },
              "inputArtifacts": [
                {
                  "name": "CalculatorFrontendService-SourceArtifact"
                }
              ],
              "configuration": {
                  "Extract": "true", 
                  "BucketName": "jrdalino-calculator-frontend"
              }
            }
          ]
        }
      ],
      "artifactStore": {
        "type": "S3",
        "location": "jrdalino-calculator-frontend-artifacts"
      }
  }
}
```

### Step 7.6: Create a pipeline in CodePipeline
```
$ aws codepipeline create-pipeline \
--cli-input-json file://~/environment/calculator-frontend/aws-cli/code-pipeline.json
```

### Step 7.7: Make a small code change, Push and Validate changes

### (Optional) Clean up
```
$ aws codepipeline delete-pipeline --name CalculatorFrontendServiceCICDPipeline
$ rm ~/environment/calculator-frontend/aws-cli/code-pipeline.json
$ aws s3api delete-bucket-policy --bucket jrdalino-calculator-frontend-artifacts
$ rm ~/environment/calculator-frontend/aws-cli/artifacts-bucket-policy.json
$ aws s3 rm s3://jrdalino-calculator-frontend-artifacts --recursive
$ aws s3 rb s3://jrdalino-calculator-frontend-artifacts --force
```

# ************************************************************

## Module 8: Install Helm

### Step 8.1: Install Helm CLI
```
$ cd ~/environment
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod +x get_helm.sh
$ ./get_helm.sh
```

### Step 8.2: Configure Helm access with RBAC
```
cat <<EoF > ~/environment/rbac.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EoF
```

### Step 8.3: Apply the config
```
$ kubectl apply -f ~/environment/rbac.yaml
```

### Step 8.4: Install helm and tiller into the cluster which gives it access to manage resources in your cluster.
```
$ helm init --service-account tiller
```

### (Optional) Clean up
```
$ kubectl delete -f ~/environment/rbac.yaml
$ rm ~/environment/rbac.yaml
$ rm ~/environment/get_helm.sh
```

# ************************************************************

## Module 9: Deploy Prometheus for basic monitoring

### Step 9.1: Install Prometheus
```
$ kubectl create namespace prometheus
$ helm install stable/prometheus \
    --name prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```

### Step 9.2: Check if Prometheus components deployed as expected
```
$ kubectl get all -n prometheus
```

### Step 9.3: Access the Prometheus server URL w/ kubectl port-forward and access /targets Web UI
```
$ kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```

### (Optional) Clean up
```
$ helm delete prometheus
$ helm del --purge prometheus
```

# ************************************************************

## Module 10: Deploy Grafana to create Dashboards

### Step 10.1: Install Grafana
```
$ kubectl create namespace grafana
$ helm install stable/grafana \
    --name grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword="EKS!sAWSome" \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer
```

### Step 10.2: Check if Grafana is deployed
```
$ kubectl get all -n grafana
```

### Step 10.3: Get Grafana ELB URL
```
$ export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
$ echo "http://$ELB"
```

### Step 10.4: Login using admin and password
```
$ kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Step 10.5: Create Grafana Dashboards
- Login into Grafana dashboard using credentials supplied during configuration
- You will notice that ‘Install Grafana’ & ‘create your first data source’ are already completed. We will import community created dashboard for this tutorial
- Click ‘+’ button on left panel and select ‘Import’
- Enter 3131 dashboard id under Grafana.com Dashboard & click ‘Load’.
- Leave the defaults, select ‘Prometheus’ as the endpoint under prometheus data sources drop down, click ‘Import’.
- This will show monitoring dashboard for all cluster nodes
- For creating dashboard to monitor all pods, repeat same process as above and enter 3146 for dashboard id

### (Optional) Clean up
```
$ helm delete grafana
$ helm del --purge grafana
```

# ************************************************************

## Module 11: Implement Liveness Probe Health Checks
- The API being a crucial part of the application it needs to be highly available

### Step 11.1: Configure the Probe
```
$ mkdir -p ~/environment/healthchecks
$ cat <<EoF > ~/environment/healthchecks/liveness-app.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-app
spec:
  containers:
  - name: liveness
    image: jrdalino/calculator-backend
    livenessProbe:
      httpGet:
        path: /health
        port: 5000
      initialDelaySeconds: 5
      periodSeconds: 5
EoF
```

### Step 11.2: Create the pod using the manifest
```
$ kubectl apply -f ~/environment/healthchecks/liveness-app.yaml
$ kubectl get pod liveness-app
$ kubectl describe pod liveness-app
```

### Step 11.3: Introduce a Failure to Test
```
$ kubectl exec -it liveness-app -- /bin/kill -s SIGUSR1 1
$ kubectl get pod liveness-app
$ kubectl logs liveness-app
$ kubectl logs liveness-app --previous
```

### (Optional) Clean Up
```
$ kubectl delete -f ~/environment/healthchecks/liveness-app.yaml
```

# ************************************************************

## Module 12: Implement Readiness Probe Health Checks
- The API being a crucial part of the application it needs to be highly available

### Step 12.1: Configure the Probe
```
$ cat <<EoF > ~/environment/healthchecks/readiness-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: readiness-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: readiness-deployment
  template:
    metadata:
      labels:
        app: readiness-deployment
    spec:
      containers:
      - name: readiness-deployment
        image: alpine
        command: ["sh", "-c", "touch /tmp/healthy && sleep 86400"]
        readinessProbe:
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 5
          periodSeconds: 3
EoF
```

### Step 12.2: Create a deployment to test readiness probe
```
$ kubectl apply -f ~/environment/healthchecks/readiness-deployment.yaml
$ kubectl get pods -l app=readiness-deployment
```

### Step 12.3: Confirm that all the replicas are available to serve traffic when a service is pointed to this deployment
```
$ kubectl describe deployment readiness-deployment | grep Replicas:
```

### Step 12.4: Introduce a Failure
```
$ kubectl exec -it <YOUR-READINESS-POD-NAME> -- rm /tmp/healthy
$ kubectl get pods -l app=readiness-deployment
$ kubectl describe deployment readiness-deployment | grep Replicas:
```

### Step 12.5: Restore pod to Ready Status
```
$ kubectl exec -it <YOUR-READINESS-POD-NAME> -- touch /tmp/healthy
$ kubectl get pods -l app=readiness-deployment
```

### (Optional) Clean Up
```
$ kubectl delete -f ~/environment/healthchecks/readiness-deployment.yaml
```

# ************************************************************

## Module 13: Implementing Auto Scaling

### Step 13.1: Configure Horizontal Pod AutoScaler (HPA) - Deploy the Metrics Server
```
$ helm install stable/metrics-server \
    --name metrics-server \
    --version 2.0.4 \
    --namespace metrics
$ kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
```

### Step 13.2: Scale an Application with Horizontal Pod AutoScaler (HPA)
- Deploy a Sample App
```
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --expose --port=80
```

- Create an HPA Resource
```
$ kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
$ kubectl get hpa
```

- Generate load to trigger scaling
```
$ kubectl run -i --tty load-generator --image=busybox /bin/sh
$ while true; do wget -q -O - http://php-apache; done
$ kubectl get hpa -w
```

### Step 13.3: Configure Cluster AutoScaler (CA)
- Configure the Cluster Autoscaler (CA)
```
$ mkdir ~/environment/cluster-autoscaler
$ cd ~/environment/cluster-autoscaler
$ vi ~/environment/cluster-autoscaler/cluster_autoscaler.yml
```

- Change <AUTOSCALING GROUP NAME>, AWS_REGION and minimum nodes (2) and maximum nodes (8)
```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
- apiGroups: [""]
  resources: ["events","endpoints"]
  verbs: ["create", "patch"]
- apiGroups: [""]
  resources: ["pods/eviction"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["pods/status"]
  verbs: ["update"]
- apiGroups: [""]
  resources: ["endpoints"]
  resourceNames: ["cluster-autoscaler"]
  verbs: ["get","update"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["watch","list","get","update"]
- apiGroups: [""]
  resources: ["pods","services","replicationcontrollers","persistentvolumeclaims","persistentvolumes"]
  verbs: ["watch","list","get"]
- apiGroups: ["extensions"]
  resources: ["replicasets","daemonsets"]
  verbs: ["watch","list","get"]
- apiGroups: ["policy"]
  resources: ["poddisruptionbudgets"]
  verbs: ["watch","list"]
- apiGroups: ["apps"]
  resources: ["statefulsets"]
  verbs: ["watch","list","get"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["watch","list","get"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["cluster-autoscaler-status"]
  verbs: ["delete","get","update"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
        - image: k8s.gcr.io/cluster-autoscaler:v1.2.2
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 300Mi
            requests:
              cpu: 100m
              memory: 300Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --nodes=2:8:<AUTOSCALING GROUP NAME>
          env:
            - name: AWS_REGION
              value: us-east-1
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt
              readOnly: true
          imagePullPolicy: "Always"
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```

- Create an IAM Policy
```
$ test -n "$ROLE_NAME" && echo ROLE_NAME is "$ROLE_NAME" || echo ROLE_NAME is not set
```

```
$ mkdir ~/environment/asg_policy
cat <<EoF > ~/environment/asg_policy/k8s-asg-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup"
      ],
      "Resource": "*"
    }
  ]
}
EoF
$ aws iam put-role-policy --role-name $ROLE_NAME --policy-name ASG-Policy-For-Worker --policy-document file://~/environment/asg_policy/k8s-asg-policy.json
$ aws iam get-role-policy --role-name $ROLE_NAME --policy-name ASG-Policy-For-Worker
```

- Deploy the Cluster Autoscaler
```
$ kubectl apply -f ~/environment/cluster-autoscaler/cluster_autoscaler.yml
$ kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

### Step 13.4: Scale a Cluster with Cluster Auto Scaler
- Deploy a Sample App
```
cat <<EoF> ~/environment/cluster-autoscaler/nginx.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-to-scaleout
spec:
  replicas: 1
  template:
    metadata:
      labels:
        service: nginx
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx-to-scaleout
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 500m
            memory: 512Mi
EoF
kubectl apply -f ~/environment/cluster-autoscaler/nginx.yaml
kubectl get deployment/nginx-to-scaleout
```

- Scale our ReplicaSet to 10
```
$ kubectl scale --replicas=10 deployment/nginx-to-scaleout
$ kubectl get pods -o wide --watch
$ kubectl logs -f deployment/cluster-autoscaler -n kube-system
```

### (Optional) Clean Up
```
$ kubectl delete -f ~/environment/cluster-autoscaler/cluster_autoscaler.yml
$ kubectl delete -f ~/environment/cluster-autoscaler/nginx.yaml
$ kubectl delete hpa,svc php-apache
$ kubectl delete deployment php-apache load-generator
$ rm -rf ~/environment/cluster-autoscaler
```

# ************************************************************

## Module 14: Logging with ElastiSearch, Fluentd, and Kibana (EFK)
- Daily/weekly/monthly report showing the operations that have been performed during that time period

### Step 14.1: Configure IAM Policy for Worker Nodes
```
$ test -n "$ROLE_NAME" && echo ROLE_NAME is "$ROLE_NAME" || echo ROLE_NAME is not set
```

```
$ mkdir ~/environment/iam_policy
cat <<EoF > ~/environment/iam_policy/k8s-logs-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
EoF

$ aws iam put-role-policy --role-name $ROLE_NAME --policy-name Logs-Policy-For-Worker --policy-document file://~/environment/iam_policy/k8s-logs-policy.json
```

```
$ aws iam get-role-policy --role-name $ROLE_NAME --policy-name Logs-Policy-For-Worker
```

### Step 14.2: Provision an Elasticsearch Cluster
```
$ aws es create-elasticsearch-domain \
  --domain-name kubernetes-logs \
  --elasticsearch-version 6.3 \
  --elasticsearch-cluster-config \
  InstanceType=t2.small.elasticsearch,InstanceCount=1 \
  --ebs-options EBSEnabled=true,VolumeType=standard,VolumeSize=20 \
  --access-policies '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Principal":{"AWS":["*"]},"Action":["es:*"],"Resource":"*"}]}'
```

```
$ aws es describe-elasticsearch-domain --domain-name kubernetes-logs --query 'DomainStatus.Processing'
```

### Step 14.3: Deploy Fluentd
```
$ mkdir ~/environment/fluentd
$ cd ~/environment/fluentd
$ vi ~/environment/fluentd/fluentd.yml
```

- Replace REGION and CLUSTER_NAME
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: fluentd
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: fluentd
  namespace: kube-system
rules:
- apiGroups: [""]
  resources:
  - namespaces
  - pods
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: fluentd
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: fluentd
subjects:
- kind: ServiceAccount
  name: fluentd
  namespace: kube-system
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
  namespace: kube-system
  labels:
    k8s-app: fluentd-cloudwatch
data:
  fluent.conf: |
    @include containers.conf
    @include systemd.conf

    <match fluent.**>
      @type null
    </match>
  containers.conf: |
    <source>
      @type tail
      @id in_tail_container_logs
      @label @containers
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag *
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <label @containers>
      <filter **>
        @type kubernetes_metadata
        @id filter_kube_metadata
      </filter>

      <filter **>
        @type record_transformer
        @id filter_containers_stream_transformer
        <record>
          stream_name ${tag_parts[3]}
        </record>
      </filter>

      <match **>
        @type cloudwatch_logs
        @id out_cloudwatch_logs_containers
        region "#{ENV.fetch('REGION')}"
        log_group_name "/eks/#{ENV.fetch('CLUSTER_NAME')}/containers"
        log_stream_name_key stream_name
        remove_log_stream_name_key true
        auto_create_stream true
        <buffer>
          flush_interval 5
          chunk_limit_size 2m
          queued_chunks_limit_size 32
          retry_forever true
        </buffer>
      </match>
    </label>
  systemd.conf: |
    <source>
      @type systemd
      @id in_systemd_kubelet
      @label @systemd
      filters [{ "_SYSTEMD_UNIT": "kubelet.service" }]
      <entry>
        field_map {"MESSAGE": "message", "_HOSTNAME": "hostname", "_SYSTEMD_UNIT": "systemd_unit"}
        field_map_strict true
      </entry>
      path /run/log/journal
      pos_file /var/log/fluentd-journald-kubelet.pos
      read_from_head true
      tag kubelet.service
    </source>

    <source>
      @type systemd
      @id in_systemd_kubeproxy
      @label @systemd
      filters [{ "_SYSTEMD_UNIT": "kubeproxy.service" }]
      <entry>
        field_map {"MESSAGE": "message", "_HOSTNAME": "hostname", "_SYSTEMD_UNIT": "systemd_unit"}
        field_map_strict true
      </entry>
      path /run/log/journal
      pos_file /var/log/fluentd-journald-kubeproxy.pos
      read_from_head true
      tag kubeproxy.service
    </source>

    <source>
      @type systemd
      @id in_systemd_docker
      @label @systemd
      filters [{ "_SYSTEMD_UNIT": "docker.service" }]
      <entry>
        field_map {"MESSAGE": "message", "_HOSTNAME": "hostname", "_SYSTEMD_UNIT": "systemd_unit"}
        field_map_strict true
      </entry>
      path /run/log/journal
      pos_file /var/log/fluentd-journald-docker.pos
      read_from_head true
      tag docker.service
    </source>

    <label @systemd>
      <filter **>
        @type record_transformer
        @id filter_systemd_stream_transformer
        <record>
          stream_name ${tag}-${record["hostname"]}
        </record>
      </filter>

      <match **>
        @type cloudwatch_logs
        @id out_cloudwatch_logs_systemd
        region "#{ENV.fetch('REGION')}"
        log_group_name "/eks/#{ENV.fetch('CLUSTER_NAME')}/systemd"
        log_stream_name_key stream_name
        auto_create_stream true
        remove_log_stream_name_key true
        <buffer>
          flush_interval 5
          chunk_limit_size 2m
          queued_chunks_limit_size 32
          retry_forever true
        </buffer>
      </match>
    </label>
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd-cloudwatch
  namespace: kube-system
  labels:
    k8s-app: fluentd-cloudwatch
spec:
  template:
    metadata:
      labels:
        k8s-app: fluentd-cloudwatch
    spec:
      serviceAccountName: fluentd
      terminationGracePeriodSeconds: 30
      # Because the image's entrypoint requires to write on /fluentd/etc but we mount configmap there which is read-only,
      # this initContainers workaround or other is needed.
      # See https://github.com/fluent/fluentd-kubernetes-daemonset/issues/90
      initContainers:
      - name: copy-fluentd-config
        image: busybox
        command: ['sh', '-c', 'cp /config-volume/..data/* /fluentd/etc']
        volumeMounts:
        - name: config-volume
          mountPath: /config-volume
        - name: fluentdconf
          mountPath: /fluentd/etc
      containers:
      - name: fluentd-cloudwatch
        image: fluent/fluentd-kubernetes-daemonset:v1.1-debian-cloudwatch
        env:
          - name: REGION
            value: us-east-1
          - name: CLUSTER_NAME
            value: calculator-eksctl
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: config-volume
          mountPath: /config-volume
        - name: fluentdconf
          mountPath: /fluentd/etc
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: runlogjournal
          mountPath: /run/log/journal
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: fluentd-config
      - name: fluentdconf
        emptyDir: {}
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: runlogjournal
        hostPath:
          path: /run/log/journal
```

- Apply
```
$ kubectl apply -f ~/environment/fluentd/fluentd.yml
$ kubectl get pods -w --namespace=kube-system
```

### Step 14.4: Configure CloudWatch Logs
```
$ cat <<EoF > ~/environment/iam_policy/lambda.json
{
   "Version": "2012-10-17",
   "Statement": [
   {
     "Effect": "Allow",
     "Principal": {
        "Service": "lambda.amazonaws.com"
     },
   "Action": "sts:AssumeRole"
   }
 ]
}
EoF

$ aws iam create-role --role-name lambda_basic_execution --assume-role-policy-document file://~/environment/iam_policy/lambda.json

$ aws iam attach-role-policy --role-name lambda_basic_execution --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

- CloudWatch Logs Console
- Select the log group /eks/calculator-eksctl/containers. 
- Click on Actions and select Stream to Amazon ElasticSearch Service.
- Select the ElasticSearch Cluster kubernetes-logs and IAM role lambda_basic_execution
- Click Next
- Select Common Log Format and click Next
- Review the configuration. Click Next and then Start Streaming

### Step 17.5 Configure Kibana
- In Amazon Elasticsearch console, select the kubernetes-logs under My domains
- Open the Kibana dashboard from the link. After a few minutes, records will begin to be indexed by ElasticSearch. 
- You’ll need to configure an index patterns in Kibana.
- Set Index Pattern as cwl-* and click Next
- Select @timestamp from the dropdown list and select Create index pattern
- Click on Discover and explore your logs

### (Optional) Clean Up
```
$ cd ~/environment
$ kubectl delete -f ~/environment/fluentd/fluentd.yml
$ rm -rf ~/environment/fluentd/
$ aws es delete-elasticsearch-domain --domain-name kubernetes-logs
$ aws logs delete-log-group --log-group-name /eks/calculator-eksctl/containers
$ aws logs delete-log-group --log-group-name /aws/lambda/LogsToElasticsearch_kubernetes-logs
$ rm -rf ~/environment/iam_policy/
```

# ************************************************************

## Security / Management / Compliance / Governance TODO's

### TODO: Add Governance - AWS Organizations and AWS Account Hardening
- Enable AWS Organizations + Landing Zone (https://aws.amazon.com/solutions/aws-landing-zone/)
- Add SCP Policies
- Create new AWS Account for product and environment
- New Email alias, AWS Account, Contacts
- Secure Root User, MFA
- Delete Default VPC's
- S3 Buckets for Logging
- S3 Buckets for ELB Logging
- S3 Buckets for Cloudfront Logs
- Enable CloudTrail Logs (Auditing)
- Enable VPC Flow Logs
- Enable ELB Logging
- Enable Config for AWS resource config tracking
- Enable SNS topics for alertning and notifications
- Enable Guard Duty for Intelligent threat detection
- Enable Security Hub for Compliance Scanning
- Wait for https://aws.amazon.com/controltower/

### TODO: Add Infrastructure as Code Templates (Custom VPC with Public and Private Subnets)
- Terraform
- Locking and Isolating State Files on S3 using https://github.com/gruntwork-io/terragrunt
- Reusable Infrastructure w/ Terraform Modules
- Testing using Terratest https://github.com/gruntwork-io/terratest

### TODO: Add Testing for Infrastruture as Code

### TODO: Setup CI/CD for Infrastructure as Code Templates
- CI/CD Pipeline using S3, CodeBuild, CodePipeline, DynamoDB for State Locking

### TODO: Add API Gateway in front of EKS Endpoint
- AWS API Gateway
- Authentication: Resource Policies/ IAM / Lambda Authorizers (token based, request parameter based)/ Cognito User Pools
- Authorization
- Enable CORS
- Swagger Documentation

### TODO: Add Web Application Firewall in front of API Gateway
- AWS Web Application Firewall
- AWS Shield
- https://github.com/aws-samples/aws-waf-sample/blob/master/waf-owasp-top-10/owasp_10_base.yml

### TODO: Add Testing - Backend Static Code Analyzer
- Find OSS alternative for SonarQube / Fortify / Checkmarx

### TODO: Add Testing - Backend Containers Scanning
- Find OSS alternative for  Anchore Engine / Aqua Microscanner / Clair / Dagda / Twistlock

### TODO: Add Web Application Firewall in front of CloudFront CDN
- AWS Web Application Firewall and AWS Shield
- https://github.com/aws-samples/aws-waf-sample/blob/master/waf-owasp-top-10/owasp_10_base.yml

### TODO: Add Testing - Frontend Static Application
- AuditJS > https://www.npmjs.com/package/auditjs
- RetireJS > https://retirejs.github.io/retire.js > Checks npm packages and compares to CVE's
- OWASP Dependency Check > https://www.owasp.org/index.php/OWASP_Dependency_Check

### TODO: Add Testing - Frontend Dynamic Application
- OWASP ZAP Web App Pentest Tool > https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project > 

### TODO: Add Database - NoSQL (DynamoDB)

### TODO: Add Caching - DynamoDB Accelerator (DAX)

### TODO: Add Database - RDBMS (RDS Postgres)

### TODO: Add Caching - ElastiCache (Redis)

### TODO: Add Testing - Commited secrets on Git Repository
- https://github.com/awslabs/git-secrets
- https://github.com/zricethezav/gitleaks

### TODO: Add Security - AWS Secrets Management
- AWS Secrets Manager / AWS Parameter Store / Hashicorp Vault

### TODO: Add SSL/TLS using AWS Certificate Manager

### TODO: Add Register/Transfer Domain Name using Route 53

### TODO: Add Authentication using Amazon Cognito

### TODO: Add Instrumentation
- AWS X-Ray / Prometheus

### TODO: Add Alerting
- AlertManager
- PagerDuty
- Slack

### TODO: Add Service Discovery
- Hashicorp Consul / Netflix Eureka / AWS CloudMap

### TODO: Add Service Mesh
- Istio / AWS Appmesh
