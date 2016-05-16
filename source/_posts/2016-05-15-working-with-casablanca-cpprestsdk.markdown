---
layout: post
title: "Working with Casablanca (CPPRESTSDK)"
date: 2016-05-15 18:24:23 +0530
comments: true
categories: C++, Coding 
---
# Casablanca 
*C++ meets WebServices*
### Introduction
Calling a WebService and processing the response, is pretty much an easy task, especially if you are programming using Java, C# or JavaScript or any of the modern programming languages. But, recently someone asked me about how to the same using C++ and in particular with Visual C++. Though, I have worked on Visual C++ and MFC in particular, I have never tried calling a WebService from VC++. So, a quick google search on this pointed me towards Casablanca.

### What?
So, what is Casablanca?
Casablanca is a Microsoft's project for cloud-based client-server communication. Casablanca is also known as C++ REST SDK. This SDK helps C++ developers to connect to services.

### How?
How do we use C++ REST SDK?
After visiting, [Casablanca](https://github.com/microsoft/cpprestsdk) official github page, I found a tutorial for [quick start](https://github.com/microsoft/cpprestsdk). Following the tutorial, I came up with the following code:

```
#include <cpprest/http_client.h>
#include <cpprest/filestream.h>
#include <cpprest/json.h>
#include <iosfwd>
#include <iostream>
using namespace web;
using namespace web::http;
using namespace web::http::client;
using namespace std;

// Retrieves a JSON value from an HTTP request.
pplx::task<void> RequestJSONValueAsync()
{
	http_client client(U("http://date.jsontest.com/"));
    return client.request(methods::GET).then([](http_response response) -> pplx::task<json::value>
    {
        if(response.status_code() == status_codes::OK)
        {
            return response.extract_json();
        }

        // Handle error cases, for now return empty json value... 
        return pplx::task_from_result(json::value());
    })
        .then([](pplx::task<json::value> previousTask)
    {
        try
        {
            const json::value& v = previousTask.get();
            // Perform actions here to process the JSON value...
			if (!v.is_null())
			{
				for (auto iter = v.cbegin(); iter != v.cend(); ++iter)
				{
					const json::value &key = iter->first;
					const json::value &value = iter->second;

					if (value.is_object() || value.is_array())
					{
						if ((!key.is_null()) && (key.is_string()))
						{
							std::wcout << L"Parent: " << key.as_string() << std::endl;
						}
						if ((!key.is_null()) && (key.is_string()))
						{
							std::wcout << L"End of Parent: " << key.as_string() << std::endl;
						}
					}
					else
					{
						std::wcout << L"Key: " << key.as_string() << L", Value: " << value.to_string() << std::endl;
					}
				}
			}

        }
        catch (const http_exception& e)
        {
            // Print error.
            wostringstream ss;
            ss << e.what() << endl;
            wcout << ss.str();
        }
    });

}



void IterateJSONValue()
{
    // Create a JSON object.
    json::value obj;
    obj[L"key1"] = json::value::boolean(false);
    obj[L"key2"] = json::value::number(44);
    obj[L"key3"] = json::value::number(43.6);
    obj[L"key4"] = json::value::string(U("str"));

    // Loop over each element in the object. 
    for(auto iter = obj.cbegin(); iter != obj.cend(); ++iter)
    {
        const json::value &str = iter->first;
        const json::value &v = iter->second;

        std::wcout << L"String: " << str.as_string() << L", Value: " << v.to_string() << endl;
    }
}

int wmain()
{
    wcout << L"Calling RequestJSONValueAsync..." << endl;
    RequestJSONValueAsync().wait();

    wcout << L"Calling IterateJSONValue..." << endl;
    IterateJSONValue();
}
```
As shown, I have used `jsontest` for testing the json. Running this code, should generate the following output.

![Output](https://onedrive.live.com/redir?resid=A344D3A301F31A3!77019&authkey=!AM7Pxil3qNBKR5k&v=3&ithint=photo%2cpng)
### Gotchas#
* One issue that one may face while setting up a project using CPPRESTSDK is that it requires VS120 toolset. So, make sure in project -> properties, the correct platform toolset is updated in Visual Studio.

### Next Steps
This was a simple example that I have created for quick demo. But, Casablanca has lot of other features that can be used. Following are some of the important features:
* This has support for [Web Sockets](https://github.com/Microsoft/cpprestsdk/wiki/Web-Socket)
* This also supports multiple platforms including Linux and Mac.

For more features and samples visit Casablanca's official [wiki](https://github.com/Microsoft/cpprestsdk/wiki).