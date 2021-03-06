# 前端报文加密

    用户登录时，对登录密码进行加密传输

## 前端加密功能

    前端提供简单的AES对称加密算法，注意key 和后端网关配置相同，这里打包混淆后，相对安全。 
    （joolun-ui\src\store\modules\user.js、base-gateway-dev.yml）
    
 ![](././index/d1.png)
 ![](././index/d2.png)

## 后端解密功能
>使用hutool提供的工具类进行解密

    public class PasswordDecoderFilter extends AbstractGatewayFilterFactory {
        @Override
        public GatewayFilter apply(Object config) {
            return (exchange, chain) -> {
                ServerHttpRequest request = exchange.getRequest();
                URI uri = exchange.getRequest().getURI();
                String queryParam = uri.getRawQuery();
                Map<String, String> paramMap = HttpUtil.decodeParamMap(queryParam, CharsetUtil.UTF_8);
    
                String password = paramMap.get(PASSWORD);
                if (StrUtil.isNotBlank(password)) {
                    try {
                        password = decryptAES(password, encodeKey);
                    } catch (Exception e) {
                        log.error("密码解密失败:{}", password);
                        return Mono.error(e);
                    }
                    paramMap.put(PASSWORD, password.trim());
                }
    
                URI newUri = UriComponentsBuilder.fromUri(uri)
                    .replaceQuery(HttpUtil.toParams(paramMap))
                    .build(true)
                    .toUri();
    
                ServerHttpRequest newRequest = exchange.getRequest().mutate().uri(newUri).build();
                return chain.filter(exchange.mutate().request(newRequest).build());
            };
        }
    }