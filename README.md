# PanPipe
PanPipe is an application designed to do content blocking.
Users can customize the interception rules and support powerful lua script.

# Document

## Basic Struct

```lua
-- TCP Part

function dorequest(tunnel, buf)
    return buf
end

function doresponse(tunnel, buf)
    -- must take care of content length
    -- when modify http raw
    -- e.g. we replace hello with same size

    return string.gsub(buf, "olleh", "hello")
end

-- HTTP Part

function should_handle_path(tunnel, context)
    if context.path == "/" then
        return true
    else
        return false
    end
end

-- if fakehttprequest's return value is not empty, there is not real request send out.
function fakehttprequest(tunnel, context)
    return {
        code = 200,
        body = "olleh dlrow",
        headers = {
            ["DupKey"] = {
                "ValueA", "ValueB"
            },
            ["TestB"] = "ValueC"
        }
    }
end

function dohttprequest(tunnel, context)
    context:setheader("another", "value")
end

function dohttpresponse(tunnel, context)
    -- content length will be changed automatically.
    context.body = string.gsub(context.body, "dlrow", "world! " .. _prop.hellokey)
end

-- hurge chunked response

-- dohttpresponse is ignored if dohttpresponse_chunk_* methods are set.
function dohttpresponse_chunk_start(tunnel, context)
end

function dohttpresponse_chunk(tunnel, context)
end

function dohttpresponse_chunk_end(tunnel, context)
end
```

# APIs

## Global
---

| Method      | Parameters    | Return Type   | Description   |
| ------------- | ------------- | ------------- | ------------- |
| notify | message:string | void | display notification, works only ios 10+ |
| invoke | identifier:string, method:string | status:boolean, results...| invoke method provide by other tasks. |

| Property | Type | Description |
| ------------- | ------------- | ------------- |
| _prop | table | properties from the task's edit panel. |
| _S | table | shared table inside the same task. |
| module | table | module map to export method for other tasks. |
| log | table | log library, e.g. log.debug(...) |
| base64 | table | lbase64 library |
| cjson | table | lua-cjson library |
| utils | table | utils library |

### Samples
---

module define
```lua
-- identifier ppp.sample

function module.test_swap(a, b)
    return b, a
end
```

module usage in other task
```lua
local status,b,a = invoke("ppp.sample", "test_swap", 1, 2)
assert(status and b == 2 and a == 1)
```

## context
---

| Method      | Parameters    | Return Type   | Description   |
| ------------- | ------------- | ------------- | ------------- |
| setheader | key:string, value:string | void | set header by kv before request or after response, old header with the same key will be replaced. e.g. `context:setheader("KEY", "VALUE")` |
| addheader | key:string, value:string | void | set header by kv before request or after response, will merge with old header(if exist). e.g. `context:addheader("KEY", "VALUE")` |
| removeheader | key:string | void | remove header by key before request or after response. |
| chunks_gsub | src:string, dest:string | count:number | do string.gsub over current chunk and last chunk, only work inside `dohttpresponse_chunk`. |
| header_map | void | headers:table | get kv map from context, header with same key will be override. |
| header_string | void | headers:string | get headers in raw string format. |

| Property | Type | Description |
| ------------- | ------------- | ------------- |
| headers | table | get header map, if there are more than one value for the same key, the value will be value list. |
| header_list | table | get header list, format `[{key = xx, value = xx}, {key = xx, value = xx} ...]` |
| body | string | get or set the body before request or after response |
| path | string | get or set the path before request, or get the path after response |

## fan
---

| Method      | Parameters    | Return Type   | Description   |
| ------------- | ------------- | ------------- | ------------- |
| sleep | second:number | void | sleep for a while, e.g. `fan.sleep(0.5)` |

## utils
---

| Method      | Parameters    | Return Type   | Description   |
| ------------- | ------------- | ------------- | ------------- |
| gettime | void | timestamp:number | get the current timestamp in double |

## log
---

trace,debug,info,warn,error

| Method      | Parameters    | Return Type   | Description   |
| ------------- | ------------- | ------------- | ------------- |
| trace | ... | void | log trace |
| isTrace | void | boolean | is trace |
