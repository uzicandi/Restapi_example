

# 1. RestController 의 반환타입


## 1-1. 단순 문자열 반환
```
@GetMapping(value = "/getText", produces = "text/plain; charset=UTF-8")
public String getText() {
	log.info("MIME TYPE : " + MediaType.TEXT_PLAIN_VALUE);
	return "안녕하세요";
}
```



## 1-2. 객체의 반환
- `SampleVO` 클래스 필요
- `'/sample/getSample'` or `.json`
```
@GetMapping(value = "/getSample", produces = { MediaType.APPLICATION_JSON_UTF8_VALUE, MediaType.APPLICATION_XML_VALUE })
	// XML, JSON 방식의 데이터를 생성할 수 있도록 작성됨
	public SampleVO getSample() {
		return new SampleVO(112, "스타", "로드");
}
```
```
@GetMapping(value="/getSample2")
    // produce 생략 가능
public SampleVO getSample2() {
	return new SampleVO(113, "로켓", "라쿤");
}
```



## 1-3. 컬렉션 타입의 객체 반환
- 여러 데이터를 한번에 전송하기 위해 배열, 리스트, 맵 타입의 객체 전송

```
@GetMapping(value = "/getList")
public List<SampleVO> getList() {
	return IntStream.range(1, 10).mapToObj(i -> new SampleVO(i, i + "First", i + " Last")).collect(Collectors.toList());
	// getList() 는 내부적으로 1~10까지의 루프를 처리하면서 SampleVO 객체를 만들어서 List<SampleVO> 로 만들어낸다
}
```

```
@GetMapping(value = "/getMap")
public Map<String, SampleVO> getMap() {
	Map<String, SampleVO> map = new HashMap<>();
	map.put("First", new SampleVO(111, "그루트", "주니어"));
    // Map을 이용하는 경우 '키(key)'에 속하는 데이터는 XML로 변환되는 경우에 태그의 이름이 되기 때문에 문자열 지정.
	return map;
}
```



## 1-4. ResponseEntity 타입

- REST 방식으로 호출하는 경우는 화면 자체가 아니라 **데이터 자체**를 전송하는 방식으로 처리되기 때문에
데이터를 요청한 쪽에서는 <u>*정상적인 데이터인지 아닌지 구분할 수 있는 확실한 방법*</u>을 제공해야 함.

- `ResponseEntity`는 데이터와 함께 __HTTP 헤더__ 의 상태 메시지 등을 같이 전달하는 용도로 사용한다
HTTP의 상태 코드와 에러메시지 등을 함께 전달할 수 있기 때문에 받는 입장에서는 확실하게 결과를 알 수 있음.

- `'sample/check.json?height=140&weight=60'` 으로 JSON 타입 데이터를 주소에 입력해줘야 데이터 전달됨.

```
@GetMapping(value = "/check", params = { "height", "weight" })
// check() 는 반드시 'height'와 'weight'를 파라미터로 전달받는다.
    public ResponseEntity<SampleVO> check(Double height, Double weight) {
    	SampleVO vo = new SampleVO(0, "" + height, "" + weight);

		ResponseEntity<SampleVO> result = null;

		if (height < 150) {
			result = ResponseEntity.status(HttpStatus.BAD_GATEWAY).body(vo); // 502 Error
		} else {
			result = ResponseEntity.status(HttpStatus.OK).body(vo);
		}

		return result;

		
	}
```



# 2. @RestController 에서 파라미터

 `@RestController`는 기존의 @Controller에서 사용하던 일반적인 타입이나 사용자가 정의한 타입(클래스)를 사용한다. 
 여기에 __추가로 몇가지 어노테이션__ 을 사용함
 - `@PathVariable` : 일반 컨트롤러에서도 사용이 가능하지만, REST 방식에서 자주 사용함. __URL 경로의 일부를 파라미터로 사용__ 할 때 이용
 - `@RequestBody` : __JSON 데이터를 원하는 타입의 객체로 변환해야 하는 경우__ 에 주로 사용
 
 
 
 ## 2.1 @PathVariable
 
 - REST 방식에서는 __URL 내에 최대한 많은 정보를 담으려고 노력함.__
 - 예전에는 __'?'__ 뒤에 추가되는 `쿼리 스트링(Query String)`이라는 형태로 파라미터를 이용해서 전달되던 데이터들이 
 REST 방식에서는 경로의 일부로 차용되는 경우가 많음.
 - 스프링 MVC 에서는 @PathVariable 어노테이션을 이용해서 URL 상에 __경로의 일부를 파라미터로 사용__ 할 수 있음.

```
 ex. http://localhost:8080/sample/{sno}
	 http://localhost:8080/sample/{sno}/page/{pno}
```

- 위의 URL 에서 __'{}'__ 로 처리된 부분은 __컨트롤러의 메서드에서 변수__ 로 처리 가능함.
- `@PathVariable`은 '{}'의 이름을 처리할 때 사용.
- REST 방식에서는 __URL 자체에 데이터를 식별할 수 있는 정보들을 표현하는 경우가 많으므로__ 다양한 방식으로 @PathVariable 이 사용됨.

```
	@GetMapping("/product/{cat}/{pid}")
	public String[] getPath(@PathVariable("cat") String cat, @PathVariable("pid") Integer pid) {
		return new String[] { "category: " + cat, "productid: " +pid };
		// '/sample/product/bags/1234'
	}
```


## 2.2 @RequestBody

- `@RequestBody` 는 전달된 요청(request)의 `내용(body)`를 이용해서 __해당 파라미터의 타입으로 변환__ 을 요구함
- 내부적으로 HttpMessageConverter 타입의 객체들을 이용해서 다양한 포맷의 입력 데이터를 변환할 수 있음
- 대부분의 경우 **JSON 데이터를 서버에 보내서** 원하는 타입의 객체로 변환하는 용도로 사용되지만,
경우에 따라서는 원하는 포맷의 데이터를 보내고, 이를 해석해서 원하는 타입으로 사용하기도 함.
- **요청한 내용을 처리하기 때문에** 일반적인 파라미터 전달방식 사용 불가. -> `@PostMapping` 사용

```
@PostMapping("/ticket")
public Ticket convert(@RequestBody Ticket ticket) {
    log.info("convert..........ticket" + ticket);
    return ticket;
}
```


# 3. REST 방식의 테스트

- 위 처럼 `GET` 방식이 아닌 `POST` 방식으로 지정되어 있으면서 **JSON 형테의 데이터를 처리**하는 것을 브라우저에서 개발하려면 많은 시간, 노력이 들어감
- `@RestController` 를 쉽게 테스트할 수 있는 방법은 주로 **REST 방식의 데이터를 전송하는 툴**을 이용하거나,
`JUnit` 과 `spring-test`를 이용해서 테스트 하는 방식을 고려함


## 3-1. JUnit 기반의 테스트 = SampleControllerTests.java 에서 진행.
 - JUnit : 탐캣을 구동하지 않고도 컨트롤러 구동할 수 있다는 장점
 - Mac, 리눅스에서는 curl 이용.
 - chrome 앱스토어에서 `REST client` 로 검색 => body JSON 입력하여 서버로 보내기 가능




# 4. 다양한 전송방식

REST 방식의 데이터 교환에서 가장 특이한 점은 기존의 GET/POST 외에 다양한 방식으로 데이터를 전달한다는 점이다.
**HTTP 전송방식**은 아래와 같은 형태다.

| 작업 | 전송방식 |
|:---:|:---:|
|`Create`|**POST**|
|`Read`|**GET**|
|`Update`|**PUT**|
|`Delete`|**DELETE**|

**REST 방식은** URI와 같이 결합하므로 회원(member)이라는 자원을 대상으로 전송방식을 결합하면 다음과 같은 형태가 된다.

| 작업 | 전송방식 | URI
|:---:|:---:|:---:|
|`등록`|**POST**|/members/new|
|`조회`|**GET**|/members/{id}|
|`수정`|**PUT**|/member/{id} + body (json데이터 등)|
|`삭제`|**DELETE**|/members/{id}|

`POST`방식도 그렇지만, `PUT`, `DELETE` 방식은 브라우저에서 테스트하기 쉽지 않기 때문에 **JUnit** 이나 **Restlet Client** 같은 도구를 이용해서 테스트하고 개발해야 한다.
