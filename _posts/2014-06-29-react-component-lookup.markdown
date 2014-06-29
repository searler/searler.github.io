---
layout: post
title:  "React.js component lookup"
date:   2014-06-29 16:00:00
categories: react javascript
---

React.js constructs html using a function over the source data. The jsx preprocessor provides
sugar to simplify the writing of the necessary code. 

This allows great power and flexibility. Consider a use case where a server streams json representing the
state of various components. The dataset is open ended, with the order and quantity completely specified by
the server. 

The json object will specify component type specific fields, with two fixed fields to drive the ui:

* key:unique key for the component, that remains constant across different states.
* type:identifies the specific UI representation for the component. 

The following code creates UI representations for `Server` and `Workstation`, which are then
mapped to the `type` via `renderers`. 

Each object delivered over the stream updates the `Root` state, and `render` generates a 
new HTML representation of the complete state. Note that `key` is by React to locate and modify
an existing representation of the component. 

```
var Workstation = React.createClass({
	render: function (){
		return (
		  <div key={this.props.value.key}>
                  --- Workstation specific representation ---
                  </div>
		);
	}
});

var Server = React.createClass({
	render: function (){
		return (
		  <div key={this.props.value.key}>
                  --- Server specific representation ---
                  </div>
		);
	}
});

var renderers = {
	"WorkstationState" : Workstation,
	"ServerState"      : Server
}; 

var Root = React.createClass({
	getInitialState: function(){
		return {nodes : {}}
	},
	componentWillMount: function(){
		var stream = new EventSource("/stream");
		stream.addEventListener("message",this.process,false);
	},
	process : function(e) {
		var payload = JSON.parse(e.data);
		this.state.nodes[payload.key] = payload;
		this.setState({nodes: this.state.nodes});
	},
	render : function() {
		var divs =  _.map(this.state.nodes,
				function(v,k,o){
					return renderers[v.type]({value:v});
				}.bind(this));
		return <div>{divs}</div>;
	}
});

```



