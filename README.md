# oatpp-curl [![Build Status](https://dev.azure.com/lganzzzo/lganzzzo/_apis/build/status/oatpp.oatpp-curl?branchName=master)](https://dev.azure.com/lganzzzo/lganzzzo/_build?definitionId=5&branchName=master)

**oatpp-curl** is a wrapper over the libcurl. It adapts libcurl to be used as a `RequestExecutor` in the oatpp's `ApiClient`.  
It supports all features of oatpp ApiClient together with Synchronous and Asynchronous oatpp's APIs.

More about Oat++:
- [Oat++ Website](https://oatpp.io/)
- [Api Client Documentation](https://oatpp.io/docs/component/api-client)
- [Api Client Demo](https://github.com/oatpp/oatpp-examples/tree/master/ApiClient-Demo)

**Please note:**
*it is recommended to use native out-of-the-box `oatpp::web::client::HttpRequestExecutor`. It is better integrated 
with oatpp and supports additional features like connection pools, retries, streaming of large data, and custom transport substitution.*  
*oatpp-curl provided mostly for demo purposes, documentation, and rare cases where functionality of oatpp native executor is not enough.*

## Requires

libcurl installed.

## oatpp ApiClient ?

oatpp ```ApiClient``` is a mechanism which enables you to generate Web Api Clients in declarative manner.
Under the hood it uses provided ```RequestExecutor``` (ex.: ```oatpp::curl::RequestExecutor```) to perform http requests. Thus you are abstracted from the low-level http-client library implementation and can substitute any other http-client library at any time with zero code changes.  
*Roughly you may treat oatpp ```ApiClient``` as Java Retrofit for C++.*

### Example

*This example is partially taken from [oatpp-consul](https://github.com/oatpp/oatpp-consul) implementation*  

#### Declare ApiClient

Declare ```ApiClient``` for remote service using code-generation

```c++
class MyApiClient : public oatpp::web::client::ApiClient {
#include OATPP_CODEGEN_BEGIN(ApiClient)
  
  API_CLIENT_INIT(DemoApiClient)
  
  API_CALL("GET", "v1/kv/{key}", kvGet, PATH(String, key))
  API_CALL("GET", "v1/kv/{key}", kvGetInDC, PATH(String, key), QUERY(String, datacenter, "dc"))
  
  API_CALL("GET", "v1/kv/{key}?raw", kvGetRaw, PATH(String, key))
  API_CALL("GET", "v1/kv/{key}?raw&dc={dc}", kvGetRawInDC, PATH(String, key), PATH(String, datacenter, "dc"))
  
  API_CALL("PUT", "v1/kv/{key}", kvPut, PATH(String, key), BODY_STRING(String, data))
  API_CALL("PUT", "v1/kv/{key}", kvPutInDC, PATH(String, key), BODY_STRING(String, data), QUERY(String, datacenter, "dc"))
  
#include OATPP_CODEGEN_END(ApiClient)
};
```

#### Create ApiClient instance

Create MyApiClient instance and configure it to use ```oatpp::curl::RequestExecutor```

```c++
/* Create ObjectMapper for serialization of DTOs  */
auto objectMapper = oatpp::parser::json::mapping::ObjectMapper::createShared();

/* Create oatpp-curl RequestExecutor with baseUrl */
auto requestExecutor = oatpp::curl::RequestExecutor::createShared("http://localhost:8500/");

/* Instantiate MyApiClient */
auto myApiClient = MyApiClient::createShared(requestExecutor, objectMapper);
```

#### Make calls

```c++

// like that...

auto value = myApiClient->kvGetRaw("key")->readBodyToString();
OATPP_LOGD("response", "value='%s'", value->c_str());

// or like that...

auto response = myApiClient->kvPut("key", "some-value");
if(response->statusCode == 200){
  auto body = response->readBodyToString();
  if(body && body == "true") {
    OATPP_LOGD("response", "value successfully saved");
  }
}
```

## More

- [oatpp-examples/ApiClient-Demo](https://github.com/oatpp/oatpp-examples/tree/master/ApiClient-Demo) - Full example project. ApiClient to ```http://httpbin.org/``` API with Sync and Async examples.
- [oatpp-consul](https://github.com/oatpp/oatpp-consul) - oatpp-consul integration based on ```ApiClient```.
