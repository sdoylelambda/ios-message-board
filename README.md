# Message Board

A student that completes this project shows that they can:

- understand and explain the difference between PUT and POST
- use URLSession to send a PUT request to create or modify data on a REST endpoint
- use URLSession to send a POST request to create data on a REST endpoint

## Introduction

Message Board is an application that will help you solidify the differences between PUT and POST HTTP methods. It allows users to create a message thread, and add messages to the threads.

Please look at the screen recording below to know what the finished project should look like:

![](https://user-images.githubusercontent.com/16965587/43822662-fbd18ed4-9aa9-11e8-8d3b-6dd8d0f1c5fe.gif)

## Instructions

Please fork and clone this repository. This repository does not have a starter project, so create one inside of the cloned repository folder.

### Part 1 - Storyboard Layout

For reasons that will be discussed later in these instructions, we're going to start the project in the storyboard. 

#### MessageThreadsTableViewController

1. Delete the view controller that comes with the Main.storyboard.
2. Add a `UITableViewController` scene, and embed it in a navigation controller. 
3. Set the navigation controller as the initial view controller.
4. Add a `UIView` as the header view to the table view controller, then add a `UITextField` to the header view.
5. Give the navigation item a title of "λ Message Board".
6. Give the cell an identifier, and set its style to "Basic". Give it an identifier as well. 
7. Create a Cocoa Touch Subclass of `UITableViewController` called `MessageThreadsTableViewController` and set this scene's class to it.
8. Add an outlet from the text field, and an action from it as well. Set the action's event to "Did End On Exit".

#### MessageThreadDetailTableViewController

1. Add a second `UITableViewController` scene to the storyboard.
2. Create a "Show" segue from the **first table view controller's cell** to this new table view controller scene. Give the segue an identifier.
3. Set the table view's cell's style to "Subtitle". This cell will display a `MessageThread.Message` object's properties.
4. Add a navigation item to the table view controller scene. Then add a bar button item in the top right of the navigation bar. Set it's "System Item" to "Add".
5. Create a Cocoa Touch Subclass of `UITableViewController` called `MessageThreadDetailTableViewController` and set this scene's class to it.

#### MessageDetailViewController

1. Add a `UIViewController` scene to the storyboard.
2. Create a "Show" segue from the `MessageThreadDetailTableViewController`'s bar button item to this newly added view controller scene. Give the segue an identifier.
3. Add a `UITextField` to the scene. Give it a placeholder of "Enter your name:".
4. Add a `UITextView` to the scene. 
5. Constrain the text field and text view.
6. Add a navigation item to the scene.
7. Add a bar button item to the top right of the navigation bar. Set its title to "Send".
8. Create a Cocoa Touch Subclass of `UIViewController` called `MessageDetailViewController` and set this scene's class to it.
9. Create outlets from the text field and text view, and an action from the "Send" bar button item.


### Part 2 - Sending Data To The API.

We're going to set up the functionality to PUT and POST our model objects right now, then come back to fetching them later.

This application will use two model objects. These model objects will form a parent-child relationship between each other. This is a common practice when dealing with JSON from a REST API. The idea behind the relationship in the context of REST APIs is that you make a model object for each "layer" of JSON. This allows you to pick out the properties you care about using in your app individually at every level. It requires a bit more of set up than you may be used to, but it makes using the data that you get from the API in the rest of your application normal.

This is the JSON object we are going to set up and use:

![](https://user-images.githubusercontent.com/16965587/43822961-de88378c-9aaa-11e8-913e-1e751b1dd5f6.png)

Or you can use this with a JSON viewer if you want (they're the exact same):

``` JSON
{
  "695398C4-498C-40A8-AA76-CB2B20DFD9FA": {
    "identifier": "695398C4-498C-40A8-AA76-CB2B20DFD9FA",
    "messages": {
      "-LJNQ2zNeL4bwG1qpwYB": {
        "sender": "Spencer",
        "text": "This is a test to make sure the API works!",
        "timestamp": 5.55405872056162E8
      }
    },
    "title": "REST APIs"
  }
}
```

#### MessageThread and Message

1. Create a "MessageThread.swift" file, and create a `MessageThread` **class**. There is a reason for making this a class which will be explained at a point where you can see the difference it will make later on in these instructions. Adopt the `Codable` protocol.
    - **NOTE:** We are calling the class `MessageThread` because there is already a class called `Thread`. As you go along in this project, make sure you don't get the two confused and accidentally use the `Thread` class instead.
2. Create the following properties:
    - A `title` string constant.
    - An `identifier` string constant.
3. We need to create a separate model object for the messages within the thread. This goes back to the concept of making a model object for each "layer" of the JSON. You may notice that the value for the `"messages"` key in the JSON is another dictionary. Inside of the `MessageThread` class, create a struct called `Message`. 

This may seem a bit odd to nest a class inside of a class, but this is fairly common when using `Codable`. However, this may change as time goes on. `Codable` is relatively new as it was only released in Swift 4. In order to refer to this `Message` class, you must write `MessageThread.Message`.

4. In the `Message` struct, add the following:
    - A `text` string constant.
    - A `sender` string constant.
    - A `timestamp` `Date` constant.
    - An initializer that gives a default value of the current time to the `timestamp` property. **HINT:** When calling the date initializer (`Date()`), the current time is taken for its value.
    - Adopt the `Equatable` and `Codable` protocols.
5. In the `MessageThread` class, add a `messages: [MessageThread.Message]` variable.
6. Create an initializer for the `MessageThread` class. Give a default value of an empty array to the `messages` property, and a value of `UUID().uuidString` to the `identifier` property.
7. Adopt the `Equatable` protocol in the `MessageThread` class. You will need to manually implement the `==` function. **HINT:** [This article](https://www.natashatherobot.com/implementing-equatable-for-protocols-swift/) will help you get started it if you're unfamiliar with how to do it. Of course, feel free to ask your PMs for help as well.

At this point, the model objects are set up to be able to be saved to the API, but we'll have some trouble fetching them. There is one last thing to implement in order for them to be decoded correctly when fetching them from the API, but we'll implement it later. 

#### MessageThreadController

1. Create a "MessageThreadController.swift" file and a `MessageThreadController` class inside of it. 
2. Add a `messageThreads: [MessageThread] = []` variable. 
3. Add the following code snippet to the class: `static let baseURL = URL(string: "https://lambda-message-board.firebaseio.com/")!`. **NOTE:** In order to access this `baseURL`, you must call the class `MessageThreadController`, then `.baseURL` because this is a static property.

You will now make the method to create and send a `MessageThread` to the API. This method will use PUT as its HTTP method:

1. Create a function called `createMessageThread`. It should take in a `title` parameter, and have a completion closure as well. The completion closure should use `@escaping`.
2. Inside the function, initialize a new `MessageThread` object. 
3.  We need to create a `URL` using the thread's `identifier` property. This will let us put the thread at a unique location in the API. To do this, use the `appendingPathComponent` method on the `baseURL` property. (again, you will need to use the `MessageThreadController` class to access the `baseURL`.) Since we are using Firebase as the API, you must also append a `"json"` path extension (using `appendingPathExtension`) to the URL or the requests will not work at all. The full URL should look something like this: `https://lambda-message-board.firebaseio.com/695398C4-498C-40A8-AA76-CB2B20DFD9FA/.json`. Note that it doesn't matter if the last `/` is in the URL or not.
4. Create a `URLRequest` variable with the `URL` you just created. Set its `httpMethod` to `"PUT"`.
5. Encode the `MessageThread` object you just initialized using `JSONEncoder` in a do-try-catch block.
6. Perform a `URLSessionDataTask` with the `URLRequest` you just created. Handle the potential error returned in the completion closure. If there was no error, then append the `MessageThread` object to the `messageThreads` variable. Remember to call `completion`, and resume the data task.

You now need a method to create messages within a `MessageThread`. In order to do that:

1. Create a function called `createMessage`. It should take in a `MessageThread` parameter (so you can put the new message in the correct thread), `text` and `sender` string parameters, and an escaping completion closure. 
2. Inside of the function, initialize a `MessageThread.Message` object from the parameters in this function. 
3. Create a `URL` just like you did in the previous function by using the `baseURL` and the `MessageThread` parameter's `identifier`. This time, also append another path component called `"messages"`. **NOTE:** Because of the way `Codable` works by default, this string should match the name of the `MessageThread`'s array of messages property or you will run into trouble later on. After you've appended the identifier and "messages" to the `URL`, append the `"json"` path extension. This path extension **must** be at the end of the `URL`.
4. Create a `URLRequest` variable with the `URL` you just created. Set its `httpMethod` to `"POST"`.
5. Encode the `MessageThread.Message` object you just initialized using `JSONEncoder` in a do-try-catch block.
6. Perform a `URLSessionDataTask` with the `URLRequest` you just created. Handle the potential error returned in the completion closure. If there was no error, then append the `MessageThread.Message` object to the `MessageThread.Message` variable. Since the `MessageThread` is a class, you can directly append it to its array of `messages` here. Remember to call `completion`, and resume the data task.

#### MessageThreadsTableViewController

1. In the `MessageThreadsTableViewController`, add a `messageThreadController: MessageThreadController` property. Set its default value to a new instance of `MessageThreadController`.
2. Using the `messageThreadController`, implement `numberOfRowsInSection` and `cellForRowAt`. Each cell should display the title of its corresponding `MessageThread` object.
3. In the action of the text field, unwrap its `text` and call the `createMessageThread` method, passing in the unwrapped text for the new thread's title. In the completion closure of `createMessageThread`, reload the table view on the main queue.
4. Go to the `MessageThreadDetailTableViewController` and create the following varables:
    - `messageThread: MessageThread?`
    - `messageThreadController: MessageThreadController?`
5. Back in the `MessageThreadsTableViewController`, implement the `prepare(for segue: ...)` method, passing the `messageThreadController`, and the `MessageThread` that corresponds to the cell the user tapped on.

#### MessageThreadDetailTableViewController

1. In the `MessageThreadDetailTableViewController` and using the `messageThread` variable, implement the `numberOfRowsInSection`, and the `cellForRowAt` methods. The cell should display the `text` and `sender` of its corresponding `MessageThread.Message` object.
2. In the `viewDidLoad`, set the title of the view controller to the `title` of the `messageThread` object.
3. Add the `viewWillAppear` method. Inside it, simply reload the table view.
4. Go to the `MessageDetailViewController` and add the following variables:
    - `messageThread: MessageThread?`
    - `messageThreadController: MessageThreadController?`
5. Back in the `MessageThreadDetailTableViewController`, implement the `prepare(for segue: ...)` method, passing this view controller's `messageThreadController`, and the `messageThread` to the `MessageDetailViewController`.

#### MessageDetailViewController

1. In the bar button item's action in the `MessageDetailViewController`:
    - Unwrap the text from the text field and text view outlets as well as the `messageThread` property. 
    - Call the `messageThreadController's` `createMessage` method, passing in the newly unwrapped values.
    - In the completion of this function, pop the view controller on the main queue.

At this point, you should be able to test the app to see if you are able to create threads and messages, but you won't be able to fetch them when the app loads quite yet.

### Part 3 - Fetching Data From The API

Since the messages are being POSTed (thus creating a unique identifier key with the message as the value), we will run into an issue when trying to decode the fetched data from the API into `MessageThread` objects. In order to fix this, we are going to manually tell the decoder how to decode `MessageThread`s instead of letting `Codable` try to do it for us.

#### MessageThread

1. In the `MessageThread` class, add this initializer. Since we haven't covered this before now, we'll talk about each step marked with comments:
``` Swift
required init(from decoder: Decoder) throws {

 // 1
 let container = try decoder.container(keyedBy: CodingKeys.self)
      
        // 2
        let title = try container.decode(String.self, forKey: .title)
        let identifier = try container.decode(String.self, forKey: .identifier)
        let messagesDictionaries = try container.decode([String: MessageThread.Message].self, forKey: .messages)
        
        // 3
        let messages = messagesDictionaries.map({ $0.value })
        
        // 4
        self.title = title
        self.identifier = identifier
        self.messages = messages
}
```

When this initializer is implemented, `JSONDecoder` (or any `Decoder`) will use the logic in this initializer to know how to parse the JSON and where to put the information in the `MessageThread` object that is being initialized.

- Comment 1: The `container` is a `KeyedDecodingContainer` object. This object allows us to access the key-value pairs of the JSON in the decoder. We give it the `CodingKeys` enum of our `MessageThread` object so it knows which key-value pairs we want in the JSON. Notice that you never implemented a `CodingKeys` enum in this class. If you don't `Codable` will synthesize one for you based on the names of the class or struct's properties.
- Comment 2: Using the `container` object, we pull out the individual values from each key-value pair. We specify the type we expect the value to be such as `String.self` or in the case of `messageDictionaries`, it is `[String: MessageThread.Message]`. We also use the `CodingKeys` cases to specify which key-value pair to pull the value from in the JSON.
- Comment 3: As stated previously, since you are POSTing the messages, the API will create a unique identifier as the key, and use the `MessageThread.Message` as the value. This is the part that would break the decoding if we didn't implement this initializer. In order to circumvent this problem, we take each `MessageThread.Message` object and sort of discard the identifier key by mapping through the dictionaries and returning only the value, which is the actual message object.
- Comment 4: We then set the classes properties to the newly decoded properties, similar to a normal memberwise initializer.

#### MessageThreadController

In the `MessageThreadController`, create a function `fetchMessageThreads`. This method should have an escaping completion closure. Inside of the function:

  - Create a `URL` that takes the `baseURL` and appends the `"json"` path extension onto it. Again, this is only necessary when using Firebase as the API.
  - Create a `URLSessionDataTask` using the `dataTask(with: URL, ...)` convenience method. 
  - Inside of the data task's completion closure, check for an error.
  - Unwrap the data, and using a `JSONDecoder` object and a do-try-catch block, decode the JSON as `[String: MessageThread].self` into a constant called `messageThreadDictionaries`. If you are unclear as to why we decoding the JSON this way, refer back to the example JSON. At the highest level, the `MessageThreads` are the values of UUID string keys. Make sure to handle errors in the `catch` statement.
  - Create a constant called `messageThreads`. For its value, `map` the `messageThreadDictionaries` and return only the values of each dictionary. 
  - Set the class's `messageThreads` variable to the `messageThreads` constant you just made so the rest of the application can use the threads.
  - Call `completion`.

#### MessageThreadsTableViewController.

Finally, in the `MessageThreadsTableViewController`, call the `viewWillAppear` method. Inside of it, call the `messageThreadController`'s `fetchMessageThreads` method. In the completion closure, reload the table view on the main queue. This will allow the table view controller to fetch any new threads and messages made every time this table view controller appears.

## Go Further

- Add a `UIRefreshControl` to the `MessageThreadsTableViewController` that will allow the user to "Pull to refresh" the table view. This should re-fetch the message threads from the API.
- Using the `timestamp` property on the `Message` object, sort the messages chronologically. Sort the list of `MessageThreads` alphabetically.
