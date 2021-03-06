Module structure for SILPA
================================

To host your existing code on SILPA you need to do small changes to
allow SILPA framework to get some information out of your module. The
important information needed by SILPA from module is its ``instance``
and ``template`` for rendering the web page.

Returning module instance
------------------------------

First information that every module which is to be hosted using SILPA
framework requires to be able to provide its own instance using a
function called ``getInstance``.

An example code from ``soundex`` module is shown below

.. code-block:: python

   def getInstance:
      return Soundex()

Template for Rendering on Webpage
----------------------------------------

All modules to be hosted on SILPA should also provide a template which
will be rendered to user who can utilize the module from web. For this
each module should include ``templates`` folder with a html file with
same name as module. i.e. ``module.html``.

If you consider ``soundex`` module as an example, you will notice
``templates`` folder contains ``soundex.html`` content.

.. code-block:: html

	{% extends "silpa.html" %}
	{% block modulescript %}
	<script type="text/javascript">
	$(document).ready(function() {
	    $("form").submit(function(event)
	    {
	        event.preventDefault();
	        id_result = $('#result');
	        id_error = $('#errormessage');
	        id_progress = $('#progress');
	        var word1 = $("input[name=word1]", 'form').val();
	        var word2 = $("input[name=word2]", 'form').val();
	        var jsonRequest = {
	            "method" :  "soundex.compare",
	            "params" : [word1,word2],
	            "id" : ""
	        };
	        $.ajax({
	            type: "POST",
	            contentType: "application/json; charset=utf-8",
	            url: "{{ url_for(main_page + '/JSONRPC' if not main_page == '/' else main_page + 'JSONRPC') }}",
	            data: JSON.stringify(jsonRequest), 
	            dataType: "json",
	            beforeSend:function(){
	                id_result.hide();
	                id_error.hide();
	                id_progress.html("Processing. Please Wait..").show();
	            },
	            success: function(msg) {
	                id_progress.hide();
	                if(msg.result==0)
	                  id_result.html("Both words are same");
	                if(msg.result<0)
	                  id_result.html("Words does not match");
	                if(msg.result==1)
	                  id_result.html("Both words are from same language and sounds alike");
	                if(msg.result==2)
	                  id_result.html("Words are from different languages but sounds alike");
	                id_result.show();
	            },
	            error: function(msg) {
	                id_progress.hide();
	                id_error.html("Something went wrong. Please try again!").show();
	            }
	        });
	    });
	 
	    $("input[type=button]", 'form').click(function(e){
	        var id_clicked = e.target.id;
	        if(id_clicked=="button1")
	            var word= $("input[name=word1]", 'form').val();
	        else
	            var word= $("input[name=word2]", 'form').val();
	        var jsonRequest = {
	                "method" :  "soundex.soundex",
	                 "params" : [word],
	                 "id" : ""
	                 };
	        id_error = $('#errormessage');
	        id_progress = $('#progress');
	        $.ajax({
	            type: "POST",
	            contentType: "application/json; charset=utf-8",
	            url: "{{ url_for(main_page + '/JSONRPC' if not main_page == '/' else main_page + 'JSONRPC') }}",
	            data: JSON.stringify(jsonRequest),
	            dataType: "json",
	            beforeSend:function(){
	                id_error.hide();
	                id_progress.html("Processing. Please Wait..").show();
	            },
	            success: function(msg) {
	                id_progress.hide();
	                // Render it
	                if(id_clicked=="button1")
	                    $('#soundex1').html(msg.result).show();
	                else
	                    $('#soundex2') .html(msg.result).show();
	                    },
	            error: function(msg) {
	                id_progress.hide();
	                id_error.html("Something went wrong. Please try again!").show();
	            }
	        });
	    });
	});
	</script>
	{% endblock %}
	{% block content %}
	          <div class="well">
	            <h3>Indic Soundex</h3>
	            <p>Soundex Phonetic Code Algorithm Demo for Indian Languages. Supports all indian languages and English. Provides intra-indic string comparison</p>
	            <ul>
	              <li><a href="http://thottingal.in/blog/2009/07/26/indicsoundex/">Read More about the algorithm </a></li>
	              <li><a href="apis.html#soundex">Read about the JSON-RPC based APIs of SILPA for this service </a></li>
	            </ul>
	 
	            <form id="soundex_form" action="" method="post">
	              <span class="help-block">To get the soundex code for a word enter a word in the text box and press Soundex button.</span>
	              <input type="text" name="word1"/>
	              <br/>
	              <input type="button" value="Soundex" class="btn" id="button1" />
	              <br/>
	              <br/>
	              <div id="soundex1" class="alert alert-info hide"></div>
	              <input type="text" name="word2"/>
	              <br/>
	              <input type="button" value="Soundex" class="btn" id="button2" />
	              <br/>
	              <br/>
	              <div id="soundex2" class="alert alert-info hide"></div>
	              <p class="help-block">To compare two words, enter the words in the below text boxes and press Compare button.</p>
	              <input type="submit" id="compare" value="Compare" class="btn"/>
	            </form>
	            <div id="progress"></div>
	            <div id="successmessage" class="alert alert-success hide"></div>
	            <div id="errormessage" class="alert alert-error hide"></div>
	            <div id="result" class="alert alert-info hide"></div>
	            </div>
	            <hr/>
	            <div class="well">
	              <h3 name="soundex">Python Soundex API</h3>
	              This service provides indic soundex algorithm based soundex codes for a word
	              <ul>
	                <li>Method: modules.Soundex.soundex
	                  <ul>
	                    <li>arg1 : the word</li>
	                    <li>Return : The soundex code for the word</li>
	                  </ul>
	                </li>
	                <li>Method: modules.Soundex.compare
	                  <ul>
	                    <li>arg1 : first word</li>
	                    <li>arg2 : second word</li>
	                    <li>Return : 0 if both strings are same, 1 if both strngs sounds alike and from same language, 2 if strings are from different langauges but sounds alike</li>
	                  </ul>
	                </li>
	              </ul>
	              Sample usage is given below.
	              <pre class="code">>>>print silpaService.soundex.soundex("&#3349;&#3390;&#3376;&#3405;&#8205;&#3364;&#3405;&#3364;&#3391;&#3349;&#3405;")
	    &#3349;APKKBF0
	    >>>print silpaService.soundex.compare("&#3349;&#3390;&#3376;&#3405;&#8205;&#3364;&#3405;&#3364;&#3391;&#3349;&#3405;","&#2965;&#3006;&#2992;&#3021;&#2980;&#3007;&#2965;&#3021;")
	    2</pre>
	</div>
	{% endblock %}

You will notice that above template uses ``Jinja`` templating system
and extends ``silpa.html`` it also contains javascript function to
utilize JSONRPC exposed by SILPA framework to post data and fetch
result. For data input a form is provided.

.. note:: SILPA framework uses Bootstrap3 for UI designing so consider
	  using same elements and structure as SILPA to get uniformity
	  in design. Also make sure this template data gets included
	  when you package your module for pypi.
