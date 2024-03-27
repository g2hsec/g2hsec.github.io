---
layout: single
title: (bWAPP)XSS - Reflected (GET)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(AJAX-JSON)-archive/image.png)

- 이전 실습과 동일하게 JSON 형태의 검색 사이트가 존재한다.
- 해당 환경에서는 AJAX 를 사용하여 새로고침 없이 입력값이 즉시 반영되는 구조이다.

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(AJAX-JSON)-archive/image-1.png)

- 일반적인 스크립트 태그를 가지고 XSS는 통하지 않는다.

```javascript
    <script>
    
        // Stores the reference to the XMLHttpRequest object
        var xmlHttp = createXmlHttpRequestObject(); 

        // Retrieves the XMLHttpRequest object
        function createXmlHttpRequestObject() 
        {	
            // Stores the reference to the XMLHttpRequest object
            var xmlHttp;
            // If running Internet Explorer 6 or older
            if(window.ActiveXObject)
            {
                try
                {
                    xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
                }
                catch (e)
                {
                    xmlHttp = false;
                }
            }
            // If running Mozilla or other browsers
            else
            {
                try
                {
                    xmlHttp = new XMLHttpRequest();
                }
                catch (e)
                {
                    xmlHttp = false;
                }
            }
            // Returns the created object or displays an error message
            if(!xmlHttp)
                alert("Error creating the XMLHttpRequest object.");
            else 
                return xmlHttp;
        }

        // Makes an asynchronous HTTP request using the XMLHttpRequest object 
        function process()
        {
            // Proceeds only if the xmlHttp object isn't busy
            if(xmlHttp.readyState == 4 || xmlHttp.readyState == 0)
            {
                // Retrieves the movie title typed by the user on the form
                // title = document.getElementById("title").value;
                title = encodeURIComponent(document.getElementById("title").value);
                // Executes the 'xss_ajax_1-2.php' page from the server
                xmlHttp.open("GET", "xss_ajax_2-2.php?title=" + title, true);  
                // Defines the method to handle server responses
                xmlHttp.onreadystatechange = handleServerResponse;
                // Makes the server request
                xmlHttp.send(null);
            }
            else
                // If the connection is busy, try again after one second  
                setTimeout("process()", 1000);
        }

        // Callback function executed when a message is received from the server
        function handleServerResponse()
        {
            // Move forward only if the transaction has completed
            if(xmlHttp.readyState == 4)
            {
                // Status of 200 indicates the transaction completed successfully
                if(xmlHttp.status == 200)
                {
                    // Extracts the JSON retrieved from the server
                    JSONResponse = eval("(" + xmlHttp.responseText + ")");
                    // Generates HTML output
                    // var result = "";
                    // Obtains the value of the JSON response
                    result = JSONResponse.movies[0].response;
                    // Iterates through the arrays and create an HTML structure
                    //for (var i=0; i<JSONResponse.movies.length; i++)
                    //    result += JSONResponse.movies[i].response + "<br/>";
                    // Obtains a reference to the <div> element on the page
                    // Displays the data received from the server
                    document.getElementById("result").innerHTML = result;
                    // Restart sequence
                    setTimeout("process()", 1000);
                } 
                // A HTTP status different than 200 signals an error
                else 
                {
                    alert("There was a problem accessing the server: " + xmlHttp.statusText);
                }
            }
        }
```

- 해당 소스코드에 나와있는 부분이며, 이번 문제도 DOM Based XSS에 좀 더 가깝다.
- 코드를 보게되면, 이전과 동일하게 json 형태로 입력값을 출력하며

```javascript
  // Executes the 'xss_ajax_1-2.php' page from the server
                xmlHttp.open("GET", "xss_ajax_2-2.php?title=" + title, true);  
                // Defines the method to handle server responses
```

- 해당 부분을 보게되면 GET 형식으로 xss_ajax_2-2.php로 보내며 title 파라미터로 입력값을 보내게 된다.
- 하지만 사이트 내에서는 AJAX 형식을 사용하므로 보이지 않는다.

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(AJAX-JSON)-archive/image-2.png)
![그림 1-4](/assets/image/bwapp/xss/Reflected%20(AJAX-JSON)-archive/image-3.png)

- 그대로 URI에 해당 경로와 파라미터에 스크립트 태그를 사용하여, 입력값을 보내게 되면
- XSS 취약점이 발생하는 것을 확인할 수 있다.