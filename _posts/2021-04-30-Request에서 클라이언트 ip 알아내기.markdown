---
layout: post
title: "Request에서 클라이언트 ip 알아내기"
date: 2021-04-30 18:20:08 +0900
category: mybatis
---

클라이언트에서 Request를 받았을때 서블릿 환경에서는 httpServeltRequest를 통해 ip 정보를 알아내는 것이 가능하다.

```
private String getClientIp(HttpServletRequest request){
		return request.getRemoteAddr();
	}
```

하지만 서버와 클라이언트 사이에 로드밸런서 L7이나 프록시 서버등이 개입할 경우

원래의 IP는 추가적인 헤더에 저장되고 getRemoteAddr로는 원래와는 다른 IP를 얻게된다.

이 경우 Request의 헤더에 담긴 IP를 불러올 필요가 있는데 다음과 같이 수행한다.

```
public static String getClientIpAddr(HttpServletRequest request) {
    String ip = request.getHeader("X-Forwarded-For");
 
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("WL-Proxy-Client-IP");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_CLIENT_IP");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getHeader("HTTP_X_FORWARDED_FOR");
    }
    if(ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
        ip = request.getRemoteAddr();
    }

    return ip;
}
```

추가적으로 헤더에 담긴 IP값을 찾고 존재하지 않는다면 getRemoteAddr을 호출하는 코드다.

로드밸런서에서 IP가 바뀌어서 오는 바람에 추가적인 코드를 만들어야 했다...
