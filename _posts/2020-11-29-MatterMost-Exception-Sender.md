---
layout: post
title: MatterMost Webhook를 활용한 Exception Sender 만들기
subtitle: "Spring에서 발생한 Exception을 MatterMost로 받아보자."
catalog: true
author:  "girawhale"
header-style: text 
tags: [MatterMost, Webhook, Spring]
---



이전까지는 팀으로 프로젝트를 진행할 때 기능별로 분류를 했었는데, Spring을 모르는 팀원과 같이 프로젝트를 진행하게 되어 역할 분담을 프론트엔드와 백엔드로 나눠 진행하게 되었다.

#### ㅤ

그러다보니 프로젝트에서 발생한 에러를 수정하기 위해 로그를 보기 어려웠다. 그러다 컨설턴트님께서 사용하고 있는 MatterMost를 통해 에러를 받아볼 수 있다는 것을 알려주시게 되어 제작을 해보게 되었다.

#### ㅤ

기본적으로 Message Template 중 Attachments의 디자인이 가장 마음에 들어 해당 형식으로 작성하기로 했다.ㅤ

![../_images/attachments-color.png](https://docs.mattermost.com/_images/attachments-color.png)

이런 느낌...

또한 `StackTrace`도 함께 보내고 싶었는데 메세지에 모두 담으면 너무 길어져 보기 불편해 어떻게 보낼까 고민하던 중  props.card를 사용하면 Additional Info 로 활용할 수 있다고 하여 추가적으로 사용하였다

![image](https://user-images.githubusercontent.com/915956/64055959-ec0cfe80-cb44-11e9-8ee3-b64d47c86032.png)

#### ㅤ

## 사전 설정

### MatterMost Webhook 설정

![img1](..\img\in-post\mattermost-exception-sender\img1.PNG)

[메뉴] > [통합] 을 선택하고 [전체 Incoming Webhook]을 클릭한다

#### ㅤ

![img2](..\img\in-post\mattermost-exception-sender\img2.PNG)

![img3](..\img\in-post\mattermost-exception-sender\img3.PNG)

[Incoming Webhook 추가하기] 버튼을 눌러 설정한 뒤 저장

#### ㅤ

![img4](..\img\in-post\mattermost-exception-sender\img4.PNG)

그러면 결과로 URL이 나오게 되는데 해당 URL을 통해 MatterMost에 Webhook을 보낼 준비를 마치게 된다.

#### ㅤ

## Spring 준비하기

### pom.xml

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>

...

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

전달하는 객체를 Json으로 전달하기 위한 Gson 라이브러리와 코드의 간결성을 위해 lombok 라이브러리를 추가하였다.



#### ㅤ

### ControllerAdvice

``` java
@ControllerAdvice
public class GrobalControllerAdvice {
	@Autowired
	private NotificationManager notificationManager;

	@ExceptionHandler(Exception.class)
	public ResponseEntity exceptionTest(Exception e, HttpServletRequest req) {
		e.printStackTrace();
		notificationManager.sendNotification(e, req.getRequestURI(), getParams(req));

		return new ResponseEntity<>(e.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
	}

	private String getParams(HttpServletRequest req) {
		StringBuilder params = new StringBuilder();
		Enumeration<String> keys = req.getParameterNames();
		while (keys.hasMoreElements()) {
			String key = keys.nextElement();
			params.append("- ").append(key).append(" : ").append(req.getParameter(key)).append('\n');
		}

		return params.toString();
	}
}
```

각 Controller에서 처리하지 못한 에러들을 모아서 에러를 받아줄 `ControllerAdvice`객체를 만든다

에러가 발생했을 때 파악하기 쉽도록 에러가 발생한 URL과 Parameter들도 함께 받기 위해 `NotificationManager`에 이것들을 함께 보내준다

#### ㅤ



## Message 객체 만들기

### Application Properties

```yml
notification:
  mattermost:
    enabled: true # mmSender를 사용할 지 여부, false면 알림이 오지 않는다
    webhook-url: # 위의 Webhook URL을 기입
    channel: # 기본 설정한 채널이 아닌 다른 채널로 보내고 싶을 때 기입한다
    pretext: # attachments의 상단에 나오게 되는 일반 텍스트 문자
    color: # attachment에 왼쪽 사이드 컬러. default=red
    author-name: # attachment의 상단에 나오는 이름
    author-icon: # author-icon 왼쪽에 나올 아이콘의 url링크
    footer: # attachment에 하단에 나올 부분. default=현재 시간
```

메세지를 보내기 위해 Webhook 등과 기본설정들을 등록할 Application Properties에 작성해 두었다

이외에도 설정할 수 있는 다양한 부분이 있지만, 적용이 되지 않거나 나에게는 필요하지 않아 제거한 부분이 많다...

만약 추가하고 싶다면 [MatterMost Docs](https://docs.mattermost.com/developer/message-attachments.html#)를 참조해서 추가로 넣으면 될 것 같다

#### ㅤ



### MatterMostMessageDTO

``` java
public class MatterMostMessageDto {

	@Getter
	public static class Attachments {
		private Props props;
		private List<Attachment> attachments;

		public Attachments() {
			attachments = new ArrayList<>();
		}

		public Attachments(List<Attachment> attachments) {
			this.attachments = attachments;
		}

		public Attachments(Attachment attachment) {
			this();
			this.attachments.add(attachment);
		}

		public void addProps(Exception e) {
			props = new Props(e);
		}

	}

	@Getter
	@AllArgsConstructor
	@Builder
	@ToString
	public static class Attachment {
		private String channel;

		private String pretext;

		private String color;

		@SerializedName("author_name")
		private String authorName;

		@SerializedName("author_icon")
		private String authorIcon;

		private String title;

		private String text;

		private String footer;

		public Attachment addExceptionInfo(Exception e) {
			this.title = e.getClass().getSimpleName();
			StringBuilder sb = new StringBuilder(text);

			sb.append("**Error Message**").append('\n').append('\n').append("```").append(e.getMessage()).append("```")
					.append('\n').append('\n');

			this.text = sb.toString();

			return this;
		}

		public Attachment addExceptionInfo(Exception e, String uri) {
			this.addExceptionInfo(e);
			StringBuilder sb = new StringBuilder(text);

			sb.append("**Reqeust URL**").append('\n').append('\n').append(uri).append('\n').append('\n');

			this.text = sb.toString();
			return this;
		}

		public Attachment addExceptionInfo(Exception e, String uri, String params) {
			this.addExceptionInfo(e, uri);
			StringBuilder sb = new StringBuilder(text);

			sb.append("**Parameters**").append('\n').append('\n').append(params.toString()).append('\n').append('\n');

			this.text = sb.toString();
			return this;
		}

	}

	@Getter
	@NoArgsConstructor
	public static class Props {
		private String card;

		public Props(Exception e) {
			StringBuilder text = new StringBuilder();

			StringWriter sw = new StringWriter();
			e.printStackTrace(new PrintWriter(sw));
			text.append("**Stack Trace**").append("\n").append('\n').append("```");
			text.append(sw.toString().substring(0,
					Math.min(5500, sw.toString().length())) + "\n...").append('\n').append('\n');

			this.card = text.toString();
		}
	}

}

```

메세지를 전달할 DTO객체를 만들어 주었다

제일 큰 틀인 Attachments에 Attachment와 Props를 넣어주는 형식으로 작성하였다

전체 메세지가 8000자..? 정도의 제한 때문에 Props때문에 메인 메세지가 잘리는 경우가 생겨 substring을 활용해 길이를 잘라주었다

#### ㅤ

만약 Slack에 메세지를 보내고 싶다면 마크다운 형태를 변경하면 된다

MM에서의 Bold는 `**TEXT**`이지만 Slack은 `*TEXT*`이므로 하나씩 제거하고, Props가 지원되지 않아 다른 형태로 바꿔주면 될 것 같다

#### ㅤ




## Spring과 MatterMost 연동

### Notification Manager

``` java
@Component
public class NotificationManager {
	private Logger log = LoggerFactory.getLogger(NotificationManager.class);
	
	@Autowired
	private MatterMostSender mmSender;

	public void sendNotification(Exception e, String uri, String params) {
		log.info("#### SEND Notification");
		mmSender.sendMessage(e, uri, params);
	}

}
```

확장성을 위해 `NotificationManager`객체를 만들어 주었다

MatterMost에 메세지를 보내기위해 `MatterMostSender`를 등록해준다

테스트 해보았을 때 Slack도 조금 변경하면 사용가능했으므로 Slack을 사용하실 분들은 `SlackSender`를 등록해두 될 것 같다



#### ㅤ

### MatterMostSender

``` java
@Component
@RequiredArgsConstructor
public class MatterMostSender {
	private Logger log = LoggerFactory.getLogger(MatterMostSender.class);

	@Value("${notification.mattermost.enabled}")
	private boolean mmEnabled;
	@Value("${notification.mattermost.webhook-url}")
	private String webhookUrl;	

    private final RestTemplate restTemplate;
	private final MattermostProperties mmProperties;
	
	public void sendMessage(Exception excpetion, String uri, String params) {
		if (!mmEnabled)
			return;
		
		try {
			Attachment attachment = Attachment.builder()
												.channel(mmProperties.getChannel())
												.authorIcon(mmProperties.getAuthorIcon())
												.authorName(mmProperties.getAuthorName())
												.color(mmProperties.getColor())
												.pretext(mmProperties.getPretext())
												.title(mmProperties.getTitle())
												.text(mmProperties.getText())
												.footer(mmProperties.getFooter())
												.build();
			
			attachment.addExceptionInfo(excpetion, uri, params);
			Attachments attachments = new Attachments(attachment);
			attachments.addProps(excpetion);
			String payload = new Gson().toJson(attachments);

			HttpHeaders headers = new HttpHeaders();
			headers.set("Content-type", MediaType.APPLICATION_JSON_VALUE);

			HttpEntity<String> entity = new HttpEntity<>(payload, headers);
			restTemplate.postForEntity(webhookUrl, entity, String.class);

		} catch (Exception e) {
			log.error("#### ERROR!! Notification Manager : {}", e.getMessage());
		}
	
	}
```

전달 받을 메세지를 보내게 될 객체이다

MatterMost Webhook을 통해 MatterMost에 직접 메세지를 보내게 된다

#### ㅤ



## 결과

![result](..\img\in-post\mattermost-exception-sender\result.PNG)

만약 성공적으로 작성되었다면 위와 같은 메세지가 도착하게 된다

또한 ⓘ버튼을 누르게 된다면

#### ㅤ

![result-card](..\img\in-post\mattermost-exception-sender\result-card.PNG)

다음 카드가 등장하게 된다

#### ㅤ


#### ㅤ

#### Github 소스보기

[**링크**](https://github.com/girawhale/mattermost-exception-sender)

#### ㅤ



___

### REFERENCE

> https://toma0912.tistory.com/95
>
> https://cheese10yun.github.io/slack-bot-spring