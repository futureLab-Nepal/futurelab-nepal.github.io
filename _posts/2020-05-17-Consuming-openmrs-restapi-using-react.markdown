---
layout: post
title: "Consuming Openmrs Restapi using React"
date: 2020-05-17 10:56:45 +0545
author: bichar4
categories: [ehr]
tags: [react, restapi]
---
# Introduction
In this post, we are going to use the [openmrs client rest web service api](https://rest.openmrs.org/#openmrs-rest-api) in react library. This post assumes that the reader has some familiarity with reactjs and bahmni.We will basically use the endpoints of openmrs to fetch a user data, make some post request on visits and observation. In my case, I am running my bahmni in vagrant box so the address for my machine is *https://192.168.33.10/*


# Setting up react boilerplate
 First of all we create a react project boilerplate using the "[create-react-app](https://reactjs.org/docs/create-a-new-react-app.html)" command. Remove the content of "app.js" file. This time we will basically include a feature where our client app will be able to search for the patient using their name as a query paramater. The endpoint for searching person with query is demonstrated [here](https://rest.openmrs.org/#search-persons). 

# Search Input 
 Let us create a search input with a button where client can search a person. For this, let us build an input element and attach couple of handler to it.
 As per the documentation, the endpoint for searching person is `GET /person?q=john` where it fetches all non voided persons that match the search query parameter.In your app.js file, add the following code. 
 ``` javascript
import React from "react";
import "./App.css";

class App extends React.Component {
  constructor() {
    super();
    this.state = {
      text:'',
      error: null,
      isLoaded: false,
      user: []
    };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }
  handleChange(e) {

  }
  handleSubmit(e) {

  }

  componentDidMount() {

  }
  render() {
    const { error, isLoaded, user } = this.state;
    if(error){
      return <div> Error : {error}</div>
    }
      return (
          <form onSubmit={this.handleSubmit}>
            <label>Search Patient</label>
            <input
              id="search-person"
              onChange={this.handleChange}
              value={this.state.text}
            />
            <button>Search</button>
          </form>
      );
    }
  }
export default App;
```
Now, as the patient starts typing the query for patient, the text field of the state is updated using `onChange` event. Modify `handleChange(e)` method as follows.

``` javascript
 handleChange(e) {
    this.setState({text:e.target.value});
    console.log(this.state.text)
  }
```
This way, whenever you type in the input box, the text value should be displayed in your console monitor.

# Making a GET request
Now as soon as we Click the search button, the handleSumbit() handler runs to make a request. Here, we need to take notice of couple of things. First we should make sure of the end point we are going to fetch. Then we need to take care of the [authentication](https://blog.restcase.com/4-most-used-rest-api-authentication-methods/) that grants the client the authority to acess the provided resources.
As per now, only basic authentication is provided in openmrs api so we need to add encoded username and password and attach it to the `Authentication header`. For making the wep api request, we are going to use fetch api, which is a promise-based JavaScript API for making asynchronous HTTP requests in the browser similar to XMLHttpRequest (XHR).
Let us write our handler code as follows:

``` javascript 
  handleSubmit(e) {
    e.preventDefault();
    //our endpoint to search for a person
    let uri = 'https://192.168.33.10/openmrs/ws/rest/v1/person?q='+this.state.text;

    //Adding necessary header for our application
    let h = new Headers();
    h.append("Accept", "application/json");
    //Default admin username and password for bahmni
    let encoded = window.btoa("superman:Admin123");
    let auth = "Basic " + encoded;
    h.append("Authorization", auth);
    //creating a request onject as per fetch api specification
    let req = new Request(uri, {
      method: "GET",
      headers: h,
      credentials: "same-origin"
    });
    //Making a fetch request
    fetch(req)
      .then((res) => res.json())
      .then(
        (result) => {
         //display result in console
          console.log(result.results)
          this.setState({
            isLoaded: true,
            user: result.results,
          });
        },
        (error) => {
          this.setState({
            isLoaded: true,
            error,
          });
        }
      );
  }
```
# CORS ERROR ???
As per now , if you tried to submit request, the app could have broken and if you check you console for error message, then there should be error as
> Access to fetch at 'https://192.168.33.10/openmrs/ws/rest/v1/patient?q=lax' from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

This error is caused by what we know as [CORS Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). Inorder to solve this problem, there are couple of technique we could follow. Since we are just working as a client, we are unable to modify the response acess header from the server. One temporary solution is to use a cors extension in the google chrome. One point to notice is that not every extension will work. This tutorial used [this extension](https://chrome.google.com/webstore/detail/cross-domain-cors/mjhpgnbimicffchbodmgfnemoghjakai) to get work around.

Now make a request and check your console. There should be data recieved from the openmrs server.

The main problem for the above approach is that it might not work for every endpoint and every request methods.So lets fix the problem and add some permanent solution for it. For this we are going to use a middleware named [`http-proxy-middleware`](https://www.npmjs.com/package/http-proxy-middleware). We will be using  `version 0.20.0` for this tutorial.Here, we basically create a reverse proxy for our request origin to that of the server origin so that our browser will think it to coming from same origin.

Install the module in your project using following command in the terminal of you workspace.

``` javascript
 npm install --save-dev http-proxy-middleware@0.20.0
```
This should add this middleware in your workpace which is reflected in your `package.json` file. Now as per the specification provided in the documentation of `http-proxy-middleware`, create a file name `setupProxy.js` inside your src directory(along with app.json).
After that write the follwing code inside that file.

``` javascript
const proxy = require('http-proxy-middleware');

module.exports = function(app){
    app.use(
        proxy("/openmrs",{
            target:"https://192.168.33.10", // your server endpoint
            changeOrigin:true,
            secure:false 
     })
    );
};
```
Now modify the request uri of the fetch request as follows:

``` javascript 
Before:
let uri = 'https://192.168.33.10/openmrs/ws/rest/v1/person?q='+this.state.text;
```
``` javascript 
After: 
let uri = '/openmrs/ws/rest/v1/person?q='+this.state.text;
```
Delete your pre-installed  cors extension and run the file. This will fulfill your fetch request bypassing the cors policy.

# Displaying patient data in main view

Now let us modify our app.js file to create the basic info of patient in our main view rather than just in console. Lets us create the component name patient person where individual person data is recieved as a props. Create a file structure as follows inside your src folder.`src > components > persons > person.js`

Add the following code in person.js:

``` javascript
import React, { Component } from 'react';

class Person extends Component {

    render() {
        const { person } = this.props;
        return (
            <div>
                <hr />
                <div className="card">
                    <h3 className="card-header">
                        {person.display}
                    </h3>
                    <div className="card-block">
                        <p className="card-text">{person.uuid}</p>
                    </div>
                </div>
            </div>
        );
    }
}

export default Person;
```
Now in your `app.js` file, import the person component and feed the user data into person component as follows.
``` javascript
//on the top of the file 
import Person from "./components/persons/person";
```
``` javascript
//modify the  render method, 

 render() {
    const { error, isLoaded, user } = this.state;
    if(error){
      return <div> Error : {error}</div>
    }
      return (
        <div>
          <form onSubmit={this.handleSubmit}>
            <label>Search Patient</label>
            <input
              id="search-person"
              onChange={this.handleChange}
              value={this.state.text}
            />
            <button>Search</button>
          </form>
           <ul>
           {user.map((user) => (
             <Person person={user} key={user.uuid} onPersonPress={this.testClick} />
           ))}
         </ul>
         </div>
      );
    }
```

In this way, the user data is successfully retrieved. Post request of the visit,observation and encounter data in the upcoming post.