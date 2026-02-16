## ❓Spring MVC에서 외부 요청을 처리하는 흐름을 설명해보세요.

<img width="450" height="300" alt="image" src="https://github.com/user-attachments/assets/ffb6a1ab-1a28-40fa-bb89-c90c6024e04e" />

.
1. 클라이언트는 URL을 통해 요청을 전송합니다.
2. **디스패처 서블릿**은 **핸들러 매핑**을 통해 해당 요청이 어느 컨트롤러에게 온 요청인지 찾습니다.
3. **디스패처 서블릿**은 **핸들러 어뎁터**에게 요청 전달을 맡깁니다.
4. **핸들러 어뎁터**는 해당 **컨트롤러**에게 요청을 전달합니다.
5. **컨트롤러**는 요청 파라미터를 받고 필요한 비지니스로직(servcie, repository)를 호출한 뒤, 결과 데이터를 Model에 담고 반환할 뷰 이름을 리턴합니다.
6. **디스패처 서블릿**은 컨트롤러가 반환한 뷰 이름을 기반으로 **뷰 리졸버**에게 실제 뷰(JSP, Thymeleaf, JSON 등)를 찾도록 요청합니다.
7. **디스패처 서블릿**은 컨트롤러가 Model에 담아둔 데이터를 찾아 해당 **뷰**에 전달합니다.
8. **뷰**는 전달받은 데이터를 이용해 최종 HTML 또는 JSON 형태로 응답을 생성하고, 클라이언트에게 반환합니다.

**➕ MVC란?** </br>
MVC는 Model, View, Controller의 약자이며, 각 레이어 간의 기능을 구분하는데 중점을 둔 디자인 패턴입니다.
- Model : 데이터 관리 및 비지니스 로직을 처리하는 부분
- View : 비지니스 로직 처리 결과를 통해 User 인터페이스가 표현되는 구간
- Controller : 사용자의 요청을 처리하고 Model과 View를 중개하는 역할

**➕ 만약 @RestController 라면? = REST API** </br>
뷰 리졸버를 사용하지 않고 -> 즉, 위 6,7,8 단계가 생략되고 </br>
5단계에서 HttpMessageConverter가 자바 객체를 JSON 문자열로 자동 변환해 반환합니다.

</br>

## ❓DispatcherServlet 의 역할에 대해 설명해 주세요.
- Spring MVC에서 모든 HTTP 요청을 가장 먼저 받아 처리하는 프론트 컨트롤러로, 전체 요청 흐름의 시작점이자 진입점 역할을 합니다.
- 클라이언트로부터 들어온 요청의 URL과 HTTP Method를 기준으로, 어떤 Controller와 어떤 메서드를 실행할지 결정합니다. 즉 @Controller와 @ReqeustMapping(또는 @GetMapping, @PostMapping)이 붙은 메서드를 찾아 매칭합니다.


**➕ 여러 요청이 들어온다고 가정할 때, DispatcherServlet은 한번에 여러 요청을 모두 받을 수 있나요?** </br>
요청 자체는 DispatcherServlet 하나가 받지만, 요청을 처리하는 스레드는 여러개가 동시에 실행되어 여러 요청을 받을 수 있습니다. Spring MVC는 기본적으로 Tomcat의 요청 스레드풀 기반으로 동작하는데, 예를들어 100개의 요청이 들어오면 Tomcat이 100개의 스레드를 생성하여 각 스레드 안에서 DispatcherServlet이 한 요청씩 처리합니다.

**➕ 수많은 @Controller 를 DispatcherServlet은 어떻게 구분 할까요?** </br>
DispatcherServlet은 요청을 직접 처리하거나 컨트롤러를 찾지 않습니다. 요청 처리 전반을 조율하는 역할을 합니다.
- HandlerMapping (컨트롤러 탐색기) : 애플리케이션 시작  모든 @Controller를 스캔해서 URL·HTTP Method·PathVariable·QueryString 조건 등을 기준으로 어떤 요청이 어떤 컨트롤러 메서드와 매핑되는지를 내부적으로 테이블 형태로 관리합니다. 이후 요청이 들어오면 이 정보를 바탕으로 정확한 Handler(컨트롤러 메서드)를 찾아 반환합니다.
- HandlerAdaptor (실행 담당자) : 찾아낸 Controller 메서드를 호출할 수 있는 적절한 Adapter를 선택 후 해당 메서드를 호출하여 요청을 처리합니다.

**➕ 컨트롤러를 찾았는데 왜 어뎁터를 사용할까?** </br>
스프링에는 다양한 형태의 컨트롤러가 있어 실행 방식이 모두 다릅니다.
DispatcherServlet이 이를 직접 처리하면 구조가 복잡해지고, 새로운 컨트롤러가 추가될 때마다 수정이 필요해 OCP를 위반하게 됩니다. 그래서 HandlerAdapter가 컨트롤러에 맞는 실행 방식을 담당하고, 실행 결과를 ModelAndView나 JSON 형태로 DispatcherServlet에 전달합니다.

</br>

## ❓Interceptor와 Servlet Filter에 대해 설명해 주세요.
개발을 하다 보면 여러 요청에서 공통적으로 처리해야 하는 로직이 있는데 Spring에서는 이런 중복 코드를 줄이기 위해 Filter, Interceptor, AOP 같은 기능들을 제공합니다. </br>
세가지 모두 어떤 로직을 수행하기 전이나 후에 공통 기능을 수행할 수 있도록 도와주는 역할을 합니다.

<img width="400" height="200" alt="image" src="https://github.com/user-attachments/assets/0ea2b1e5-aecb-4d54-815b-e967f845d376" />

- **필터** : DispatcherServlet에 요청이 전달되기 전/후에 동작합니다. Spring 컨테이너가 아니라 톰캣 같은 웹 컨테이너에서 관리되고, Spring의 범위 밖에서 동작하는 기술입니다. 인코딩 처리, CORS 설정, 공통 인증 검사 등 모든 요청에 대해 전역적으로 처리해야 하는 작업에 사용됩니다.
- **인터셉터** : DispatcherServlet이 Controller를 호출하기 전/후에 동작합니다. Filter와 달리 Spring 컨텍스트 내부에서 관리되고, Spring MVC 흐름 안에서 요청과 응답을 가공할 수 있습니다. 로그인 여부 검사, 권한 체크 등 Controller와 관련된 로직을 처리할 때 사용합니다.

**➕ 설명만 보면 인터셉터만 쓰는 게 나아 보이는데, 아닌가요?** </br>
두 기능은 동작 위치가 다르기 때문에 역할도 다릅니다. </br>
Filter는 가장 앞단에서 동작하기 때문에 HttpServletRequest와 Response 객체를 직접 교체할 수 있습니다. 즉, 요청과 응답 자체를 바꿔치기 할 수 있습니다. </br>
반면 Interceptor는 DispatcherServlet이 관리하기 때문에 Request/Response 객체를 교체할 수는 없고, 기존 객체에 대한 검사나 속성 추가 정도만 가능합니다.



