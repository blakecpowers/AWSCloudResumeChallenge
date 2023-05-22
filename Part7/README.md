# Request Endpoints to Implement Visitor Count
In this lesson, we will build a web application that tracks and displays the number of visitors to a web page. Using JavaScript and AWS services, we will make API requests, store data in DynamoDB, and create a responsive frontend. By the end, you will have a solid understanding of serverless web development and working with AWS infrastructure.


## Contents
1. [Javascript](#javascript)
2. [Database](#database)
3. [Put and Get Lambda Methods](#put-and-get-lambda-methods)
4. [Insert Data into DynamoDB](#insert-data-into-dynamodb)
5. [Get Data from DynamoDB](#get-data-from-dynamodb)
6. [Demo](#demo)
7. [Conclusion](#conclusion)

## Javascript
First, we'll add the following javascript code as a script tag in our HTML file. This will fetch our API and inject the response from our API into our page (so we can see it).
![JSCodeP7](/images/JSCodeP7.png)

A simple HTML file with the JS code would look something like this:

![htmlJSP7](/images/htmlJSP7.png)

You could use this method, or a better approach would be to create a separate JS file whrere the script tag is stored. Since our script tag is super simple, in this case we could just put it into the HTML page. 

## Database
For our database, we'll be using DynamoDB. A fully managed NoSQL database service provided by Amazon Web Services (AWS). It is designed to provide fast and scalable storage for applications that require low-latency access to data. It also offers a flexible schema, allowing you to store and retrieve data without the need for predefined table schemas or fixed column lengths. Automatically scaling to handle high read and write traffic.

Next, we set up DynamoDB with Cloudformation with our AWS SAM template.


![dynamodbtemplate](/images/dynamodbtemplate.png)

We set the table name, billing mode (on demand billing each time a request comes through), and our key (which is an ID).

Once the following has been added to our SAM template, we can run:

```
make deploy-infra
```

Now it's been deployed and AWS will begin creating that DynamoDB resource. We can see here that our Cloudformation stack is updating.

![cfupdatep7](/images/cfupdatep7.png)

We can navigate to the events on this stack to see the create (for dynamoDB) is in progress. 

![cfinprog](/images/cfinprog.png)

Next we can navigate to DynamoDB in AWS and see the table that was created

![dynamodbtable](/images/dynamodbtable.png)



# Put and Get Lambda Methods
In this section, we'll be creating two lambda functions. One for getting objects from DynamoDB and another for putting objects into dynamoDB. Our previous function had an API setup where we just returned a single static count. This section will break up this lambda function into two separate lambda functions. One for adding values to the DB and another to get values from the DB. 

Let's breifly cover "why" we're breaking these into two lambda functions. There's two architectural patterns we could take here. Monolithic functions or a smaller functions where you have one lambda function per piece of functionality. Each pattern could be a better fit for different use cases. 

- Monolithic: All of your code in a single lambda deploy. Requests will come through your API Gateway and hit the same lambda function each time. There, will be some "if" condition logic that will handle requests appropriately. 

- Lambda per Functionality, which we'll be implementing here in this section. 

First, let's rename the "HelloWorld" lambda function before to "GetFunction" in the template.yaml file.

![helloworldp7](/images/helloworldp7.png)

![getp7](/images/getp7.png)


We also create a "PutFunction" that will represent our other lambda function for putting objects into DynamoDB.

![putp7](/images/putp7.png)

Note: This has a GET HTTP request method still. We'll clean this up in a later lesson. 

Our file structure naming will also need to change since it's no longer called "hello-world" but rather "get-function". I'll attach the full file paths in this lesson. 

Once we deploy this, this will create two different lambda functions. 

```
make deploy-infra
```

We can navigate to lambda in AWS to see the following: 

![awslambdap7](/images/awslambdap7.png)


# Insert Data into DynamoDB
Here we'll be adding the put logic into our lambda function which inserts items into our database. This includes Docker, IAM, and AWS SDK to interact with DynamoDB. 

Let's take a look at how we use the AWS SDK in our lambda put function. Note: All of the complete associated code will be attached in this lesson. 

```
func handler(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {

	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	svc := dynamodb.New(sess)

	input := &dynamodb.UpdateItemInput{
		TableName: aws.String("cloud-resume-challenge"),
		Key: map[string]*dynamodb.AttributeValue{
			"ID": {
				S: aws.String("visitors"),
			},
		},
		UpdateExpression: aws.String("ADD visitors :inc"),
		ExpressionAttributeValues: map[string]*dynamodb.AttributeValue{
			":inc": {
				N: aws.String("1"),
			},
		},
	}

	_, err := svc.UpdateItem(input)

	if err != nil {
		log.Fatalf("Got error calling UpdateItem: %s", err)
	}

	return events.APIGatewayProxyResponse{
		Headers: map[string]string{
			"Access-Control-Allow-Origin":  "*",
			"Access-Control-Allow-Methods": "*",
			"Access-Control-Allow-Headers": "*",
		},
		StatusCode: 200,
	}, nil
}

func main() {
	lambda.Start(handler)
}
```

Let's talk through this code briefly. At the top we setup a new AWS session and configures it to use the shared AWS configurtation shared on the local system.

The "dynamodb.UpdateItemInput" is the part where we update dynamodb and add in our item. First we access the correct table, access the key with ID "visitors", and an expression to add visitors "1".

Below is some error handling and lastly some CORS configuring. 

We've also added a new command to the make file. This will build the code using sam locally, which will build the application, set uo the necessary environment, trigger the execution of the "PutFunction" locally. Allowing us to test and debug the function locally before deploying it to the cloud. 

```
invoke-put:
	sam build && aws-vault exec acloudguru-sandbox --no-session -- sam local invoke PutFunction
```

Notice we're also putting our AWS credentials "using aws-vault acloudguru..." to the local sam invoke function. The reason for that is so that our lambda has access to the DynamoDB table. However, when we deploy this, by default lambda won't have access to DynamoDB. So, what we do is apply a policy to the template.yaml file.

```
`Policies:
  - DynamoDBCrudPolicy:
      TableName: cloud-resume-challenge
```

This applies an out of the box AWS SAM policy called "DynamoDBCrudPolicy" to the DynamoDB table.


![IAMDynamo](/images/IAMDynamo.png)

We can see that our IAM policy has now applied this new role policy. This allows us to run our lambda function to access dynamo db (without passing credentials in prod like we do locally). 


# Get Data from DynamoDB
Similarly to above, we're using the AWS SDK again, but this time to get data from DynamoDB. Here's the sample code from our get lambda fucntion.

```
func handler(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {

	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	svc := dynamodb.New(sess)

	var input = &dynamodb.GetItemInput{
		TableName: aws.String("cloud-resume-challenge"),
		Key: map[string]*dynamodb.AttributeValue{
			"ID": {
				S: aws.String("visitors"),
			},
		},
	}

	queryOutput, err := svc.GetItem(input)

	type Count struct {
		ID       string `json:"ID"`
		Visitors string `json:"visitors"`
	}
	count := Count{}

	err = dynamodbattribute.UnmarshalMap(queryOutput.Item, &count)

	if err != nil {
		log.Fatalf("Got error calling UpdateItem: %s", err)
	}

	return events.APIGatewayProxyResponse{
		// TODO: This is temporary, come back to fix this later
		Headers: map[string]string{
			"Access-Control-Allow-Origin":  "*",
			"Access-Control-Allow-Methods": "*",
			"Access-Control-Allow-Headers": "*",
		},
		Body:       fmt.Sprintf("{ \"count\": \"%s\" }", count.Visitors),
		StatusCode: 200,
	}, nil
}

func main() {
	lambda.Start(handler)
}
```

Again establishing a session, getting the table, and getting item where "ID" = "visitors" which will be the visitor count. Error handling. Lastly returning the count value back to the browser.  

We also need to add to the makefile a local invoke for this "get". 

```
invoke-get:
    sam build && aws-vault exec acloudguru-sandbox --no-session -- sam local invoke GetFunction
```

We also add the same policies to the GetFunction in the Template.yaml file, exactly how we did with the Put function.

The index.html script tag also needs to be updated to reflect this. 

![indexp7](/images/indexp7.png)

Which does the following:

1. `fetch('https://3zxynma4be.execute-api.us-east-1.amazonaws.com/Prod/put')`: This fetch request sends a GET request to the specified URL, triggering a server-side action to store or update data using the PUT method.

2. `.then(() => fetch('https://3zxynma4be.execute-api.us-east-1.amazonaws.com/Prod/get'))`: This line of code uses the `.then()` method to chain the subsequent fetch request. After the first request is completed, it sends a GET request to the specified URL to retrieve data (using the GET method).

3. `.then(response => response.json())`: This line extracts the JSON data from the response of the second fetch request.

4. `.then((data) => { document.getElementById('replaceme').innerText = data.count })`: This line updates the content of an HTML element with the ID 'replaceme' with the value of the 'count' property from the retrieved JSON data.

In summary, this script fetches data from an API endpoint, stores or updates data with a PUT request, retrieves data with a GET request, and updates the content of a specified HTML element with the retrieved data's count value.


# Demo
Now, we can navigate to the S3 bucket url (which is hosting our site), refresh, and see the count increase.

![refresh1](/images/refresh1.png)

![refresh2](/images/refresh2.png)





## Conclusion
In this lesson, we successfully built a web application that tracks visitor counts. We leveraged JavaScript to make API requests and update the page content dynamically. AWS services played a crucial role in our application's backend infrastructure, with API Gateway acting as the interface to our Lambda functions, and DynamoDB providing a reliable and scalable database for storing the visitor count.
