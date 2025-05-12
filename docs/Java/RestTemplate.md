## Spring Boot 配置 RestTemplate 跳过证书校验
```java
@Bean  
@Primary  
RestTemplate restTemplate() {  
    return new RestTemplate();  
}  
  
@Bean  
public RestTemplate unTrustedRestTemplate() throws NoSuchAlgorithmException, KeyManagementException {  
    SSLContext sslContext = SSLContext.getInstance("TLS");  
    sslContext.init(null, new TrustManager[]{new X509TrustManager() {  
        @Override  
        public void checkClientTrusted(X509Certificate[] chain, String authType) {  
        }  
  
        @Override  
        public void checkServerTrusted(X509Certificate[] chain, String authType) {  
        }  
  
        @Override  
        public X509Certificate[] getAcceptedIssuers() {  
            return new X509Certificate[0];  
        }  
    }}, new java.security.SecureRandom());  
  
    // 创建自定义的 RequestFactory    
    SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory() {  
        @Override  
        protected void prepareConnection(HttpURLConnection connection, String httpMethod) throws IOException {  
            super.prepareConnection(connection, httpMethod);  
  
            if (connection instanceof javax.net.ssl.HttpsURLConnection) {  
                // 跳过证书校验  
                HttpsURLConnection httpsConnection = (HttpsURLConnection) connection;  
                httpsConnection.setHostnameVerifier((hostname, session) -> true);  
                httpsConnection.setSSLSocketFactory(sslContext.getSocketFactory());  
            }  
        }  
    };  
  
    return new RestTemplate(requestFactory);  
}
```