```
  location / {
    lua_need_request_body on;
    access_by_lua_block {
      local request_path = ngx.var.request_uri

      local requests_limit = 3
      local time_limit = 3600
      local shared_dict = "register_dict"

      local fingerprints = {
        {
          type = "remote_ip",
          value = ngx.var.remote_addr
        }
      }

      local request_headers = ngx.req.get_headers()

      if request_headers["X-API-Key"] ~= nil then
        fingerprints[#fingerprints + 1] = {
            type = "api_key",
            value = request_headers["X-API-Key"]
          }
      end

      limit_spam_requests(request_path, shared_dict, requests_limit, time_limit, fingerprints)
    }
```

Основные переменные:
* `request_path` - путь запроса - можно выставить один для разных блоков и тем самым объеденить блокировки на разных url
* `requests_limit` - лимит запросов за определенное время
* `time_limit` - время, в течение которого разрешено определенное количество запросов
* `fingerprints` - основной параметр, который указыает какие параметры для проверки запросов использовать. Сюда можно добавить любые параметры из json или из get/post аргументов.
