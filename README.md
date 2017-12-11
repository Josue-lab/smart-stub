## Smart Stub version 0.9.13

The Smart Stub provides some powerful capabilites:
* built-in swagger validation
* built-in state transition validation
* ability to prime responses
* ability to globally substitute strings in all responses

### Build instructions

sbt reload clean assembly   // build and run tests

### Usage
Engineers should create an Object (e.g. MyStub.scala) which extends SmartStub and then:
* override setUpMocks() with your stubb mappings
* add a driver method
* add a swagger json under resources
* add a state model under resources



### Example
The project contains StubTests.scala which offers an example of mocking and test using SmartStub.

### A driver method
```
   def main(args : Array[String]) {
      // port, canned responses directory, swagger json file, state model file, opening state
      MyStub.configureStub(args(0).toInt, args(1), args(2), args(3), "registered")
      MyStub.start
   }
```
### Wiremock mappings
```
    override def setUpMocks(cannedResponsesPath: String): Unit  = {

      wireMockServer.stubFor(post(urlMatching(".*/cars"))
             .withRequestBody(equalToJson("{\"action\":\"drive\"}",true,true))
             .willReturn(
               aResponse()
                 .withTransformerParameter("nextState", "moving")
                 .withBody("""{"response":"hello"}""")
                 .withStatus(200)));
    }
    
``` 
    The optional withTransformerParameter() method will indicate the state this action should place you in
    If you are not interested in state, use withTransformerParameter("nextState", "any") or omit the line.
    



### Prime next response

    If you want to replace the whole response with your own you can tell Smart Stub by Posting a primed response
    as follows:
    
        doPost(url+MyStub.PRIMED_RESPONSE_URL+"?"+MyStub.RESPONSE_STATUS_QUERY_PARAM+"=200",  """{"response":"hello"}""")
    
    The next call to this stub will return this request's body and headers and the passed status
    
    You can create a series of responses by calling this multiple times and the stub will read FIFO
    
    Of course, if you have a wiremock handle, you can  add mock stubs on the fly using the wiremock API.
    
    The alternative approach to priming is to rely on canned responses from the stub. The canned approach
     would benefit from a documented and distrubuted list of known canned identifiers (users, products etc)
     so that testing can lever consistency between separate stubs.
     
    If you just want to tweak the canned response:
    - make a call to the stub service or access the canned json in the jar
    - tweak the response payload
    - prime the next response with the tweaked payload
    

### String substitute


    If you only want to globally tweak something in the canned response you can tell Smart Stub to perform a
    simple replaceAll for a given string for all responses. This would be useful, for example, if you want
    to use a username diffferent to the one in the canned response.

        doGet(url+MyStub.RESPONSE_SUBSTITUTE_STRING_URL+"?"+MyStub.RESPONSE_SUBSTITUTE_TARGET_QUERY_PARAM+"=<from>&"+MyStub.RESPONSE_SUBSTITUTE_WITH_QUERY_PARAM+"=<to>")
    

### State model

The state model json will be of the following format:
```
   {
     "stateTransitions": [
         {
           "prestate": "<some state>",
           "poststate": "<some state>"
         },
         {
           "prestate": "<some state>",
           "poststate": "<some state>"
         }
     ]
   }
```
## Errors
* An invalid request payload will return a 400 with an error and an "Invalid contract" message
* An invalid response (from the stub) will also result in a 400 with an error and an "Invalid contract" message
* An invalid state transition will (you guessed it) return a 400 with an error and an "Invalid state transition" message

## License

This project is licensed under the APACHE License - see the LICENSE file for details

## Acknowledgments

The awesome Wiremock team
