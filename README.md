# Rasa Action Server on Node-Red
[![DOI](https://zenodo.org/badge/334833949.svg)](https://zenodo.org/badge/latestdoi/334833949)

Rasa action server on Node-Red (or short ``rasaas`` is a set of nodes that can be used to implement an action server for Rasa-based chatbots. 

## Rasa
Rasa is a platform for building chatbots. A dialogue with a Rasa chatbot consists of a series of intents uttered by the user and actions by which the chatbot responds to the intents.

Rasa chatbots can serve as conversational assistants, meaning, users give orders in natural language, which the assistant fulfills in the background.
The chatbot delegates actions, which it is not able to perform by built-in means, to action servers. In Rasa terminology, these actions are termed custom actions, as they have to be implemented when building the conversational assistant.
An action server is a system component which manages the fulfillment of custom actions. Rasa brings a Python SDK that helps to code and setup action servers.

Rasa action servers run as stand-alone servers and communicate with the chatbot via rest calls. Most often, fulfilling user orders involves calling further services. Here, a platform like Node-Red, which specializes in wiring together services, APIs and even hardware devices, can be useful.  

## Node-Red
Node-Red is a low-code platform running on Node.js. With Node-Red, you get a browser-based GUI where you can create flows by drag-and-dropping nodes from a palette onto the flow editor. 
There are several core nodes which come  with Node-Red natively as well as a wealth of additional  nodes that are contributed by numerous sources. 

You create a flow by connecting nodes in the visual editor. A flow passes JSON objects from node to node, with the nodes manipulating the JSON payload and performing the actions for which they were built.

Among the Node-Red core nodes, there are an <code> HTTP in</code> node as well as its counterpart, an <code>HTTP response</code> node.
The  <code> HTTP in</code> node creates an HTTP endpoint. An HTTP request hitting this endpoint triggers the flow starting at the <code>HTTP in</code> node. Normally, this flow should end in an <code>HTTP response</code> that answers the HTTP request with an HTTP response carrying a payload that the intermediate nodes of the flow have built. So, when an action server for Rasa is set up on Node-Red, custom action requests can be received  by an <code> HTTP in</code>, and an <code> HTTP response</code> can serve the response to Rasa with the nodes in the flow creating this response.

Rasa uses JSON for communication with its action servers.
Node-Red has several core nodes for manipulating JSON, with or without programming in JavaScript. Additionally, it is easy to install contributed nodes that connect to a multitude of APIs, like, e.g, database APIs, and cloud service APIs.

## Rasa Action Server on Node-Red
Rasa Action Server on Node-Red implements a set of nodes that make it easy to set up an action server for Rasa chatbot on Node-Red. 
Specifically, these nodes implement the Rasa action server HTTP API as specified in <a href="https://rasa.com/docs/action-server/http-api-spec">Rasa Action Server HTTP API</a>.

Currently, there are nodes for sending button responses, text responses, images and attachments. If other types of responses are needed, these may be generated by Node-Red built-in nodes.

## How to use
### Typical structure of a flow
A Rasa Action Server on Node-Red flow starts with a Node-Red core <code>HTTP in</code> node that handles the requests sent to the action server. The flow ends with a Node-Red core <code>HTTP response </code> node answering the requests sent by a Rasa chatbot.  

The second node in Rasa Action Server on Node-Red flow should be the <code>Init</code> node. The <code>Init</code> node transforms the message object as emitted by the <code>HTTP in</code> node. It  stores additional information in the <code>flow</code> scope (instead of node scope), in particular, the HTTP request and response objects that are required by the <code>HTTP response</code> node for completing the request. Consequently, the last node in a Rasa Action Server on Node-Red  flow is an <code>HTTP response</code> node. 

The second-to-last node is the Rasa Action Server on Node-Red <code>finish</code> node. This node assembles the message object into the correct format for the <code>HTTP response</code> node.

If an action server is able the perform several actions, a <code>switch</code> node following the <code>init</code> node can route the flow to corresponding branches.

The actions of the Rasa Action Server on Node-Red like, e.g., calls to APIs or databases are implemented by Node-Red nodes, speeding up development as  pre-built Node-Red nodes for Web APIs and services can be used and wired together by visual programming.  

The results of the actions, API calls, database queries, and so on, need to be transformed into a Rasa-compatible format before sending them back to Rasa chatbot. Specifically, the Rasa chatbot expects as a result responses that it utters in its dialog with a user.


### Nodes generating responses
Currently, Rasa Action Server on Node-Red has three nodes that give  support for these transformations: 
- <code>sendtext</code>  generates a simple text response,
- <code>sendbuttons</code>  generates a response that generates buttons to be shown to the user,
- <code>sendextra</code>  adds an image or an attachment to a response.

The nodes offer  two options for providing content (i.e., text, references to images or attachments, or buttons)  for the responses:
- specify a fixed text or content in the config of the node,
- specify in the node config how to retrieve the text or content from `msg.payload`.

A Rasa chatbot expects to receive an array of responses from the action server. The chatbot  will present the responses to the user one by one corresponding to their order in the array. 
The nodes allow to specify the position where the response should appear in the responses array.

Admittedly, the <code>send text</code> node is redundant to some extent, since the other `send` nodes can also generate text responses. Nevertheless, the `send text` node helps to make the flows clearer and more declarative.

### A node for setting slots
Slots are the memory of a Rasa chatbot. A slot is identified by a name and can store a piece of information  that the chatbot has gathered during the dialogue. 
Slots are also sent to the action server in a custom action request. Typically, slot values serve as parameters for the actions that an action server performs. E.g., a weather bot needs to know the targeted location in order to yield  correct weather information. 

Slot values can easily be read and evaluated by Node-Red nodes. 
In some cases, it is necessary to clear or set Rasa slots within a  custom action. This is supported by the <code>setslots</code> node.

### An example flow
An example flow can be found in https://github.com/weberi/rasaas-demo-flow.


### Installation
- Prerequisite: you have installed Node-Red 
- Add node-red-contrib-rasa-actionserver to Node-Red palette
- Design your flow. Start it with an ``HTTP in`` node listening at ``/webhook``
- Configure the action server endpoint in Rasa ``endpoints.yml``, e.g.,

```yaml
action_endpoint:
        url: http://127.0.0.1:1880/webhook
```

## References

  - [Rasa Action Server HTTP
    API](https://rasa.com/docs/action-server/http-api-spec)



