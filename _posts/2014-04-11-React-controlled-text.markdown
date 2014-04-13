---
layout: post
title:  "React.js controlled text"
date:   2014-04-11 17:00:00
categories: react.js
---
The react.js [documentation](http://facebook.github.io/react/docs/forms.html) describes how a text 
field can be used as a controlled field and indicates that the following code can be used to
limit the maximum length of the field to 140 characters.
{% highlight javascript %}
 getInitialState: function() {
    return {value: 'Hello!'};
  },
  handleChange: function(event) {
     this.setState({value: event.target.value.substr(0, 140)});
  },
  render: function() {
    var value = this.state.value;
    return <input type="text" value={value} onChange={this.handleChange} />;
  }
{% endhighlight %}
That is true but react.js falls to correctly manage the cursor position. That position will match 
the user input until the 140 character is reached. At that point, the cursor will jump to
the end of the field. Evidently react.js only leaves the selection untouched so long as the
html is not actually impacted by the handleChange function. This can be made even clearer by adding
toUpperCase to force the field contents to always be uppercase. The cursor position then jumps to
the end of the field on every keystroke.

This issue is referenced by several GitHub tickets [1](https://github.com/facebook/react/issues/955) [2](https://github.com/facebook/react/issues/278), StackOverflow discussions [1](http://stackoverflow.com/questions/22677931/react-js-onchange-event-for-contenteditable) [2](http://stackoverflow.com/questions/22410767/insert-at-cursor-in-react), etc.
As far as I can determine, none of these provide a functional solution.

One approach is to explicitly manage the selection
{% highlight javascript %}
var LocalEditor = React.createClass({
  getInitialState: function() {
    return {value: 'LocAl!'};
  },
  handleChange: function(event) {
    var src = event.target.value;
    var text = src.replace(/a/g,'');
    var caret = event.target.selectionStart - (src.length-text.length);
    this.setState(
      {value: text}, 
      function(){
	this.getDOMNode().setSelectionRange(caret,caret);
      });  
  },
  render: function() {
    var value = this.state.value;
    return <textarea type="text" value={value} onChange={this.handleChange}/>;
  }
});
{% endhighlight %}
This editor is designed for Canadians, dropping any 'a's that are entered.
This works as expected and there are no visual artifacts. 

Extending the above example so the string manipulation is provided by the web server
{% highlight javascript %}
var Editor = React.createClass({
  getInitialState: function() {
    return {value: 'Remote!'};
  },
  handleChange: function(event) {
    var msg = {text:event.target.value,
      caret:event.target.selectionStart};
      $.ajax({url: "/text", type: "POST",
  data: JSON.stringify(msg),
   contentType:"application/json; charset=utf-8", dataType:"json",
   success : function(response){
     this.setState({value: response.text}, 
         function(){
            this.getDOMNode().setSelectionRange(response.caret,response.caret);
         });  
	}.bind(this)});
  },
  render: function() {
    var value = this.state.value;
    return <textarea type="text" value={value} onChange={this.handleChange} />;
  }
});
{% endhighlight %}
This works. However; when entering an 'a' there is a noticeable artifact as the cursor jumps to the end
and then returns to the correct position.

It has been suggested that the change should be made to a parent state and allow the child to be re-rendered,
but that does not improve matters.
{% highlight javascript %}
var DelegatingEditor = React.createClass({
  getInitialState: function() {
    return {value: 'Remote!'};
  },
  handleChange: function(event) {
    var msg = {text:event.target.value,
      caret:event.target.selectionStart};
      $.ajax({url: "/text", type: "POST",
        data: JSON.stringify(msg),
             contentType:"application/json; charset=utf-8", dataType:"json",
             success : function(response){
               this.setState({value: response.text}, 
                   function(){
                      this.getDOMNode().setSelectionRange(response.caret,response.caret);
               });  
             }.bind(this)});
  },
  render: function() {
    return <ComponentEditor value={this.state.value} handleChange={this.handleChange}/>;
  }
});

var ComponentEditor = React.createClass({
  render: function() { 
    return <textarea type="text" value={this.props.value} 
                     onChange={this.props.handleChange} />;
  }
});
{% endhighlight %}

The implementation is thus suboptimal but still useful. 