# Javamail로 이메일 보낼 때 noname attachment 문제

자바메일을 통해 메일을 보낼 때 Background image, Logo Image 등 파일들을 Inline으로 추가해서 보낸다

```java
 MimeMessagePreparator getMimeMessagePerpetrator(Email email) {
        return mimeMessage -> {
            MimeMessageHelper message = new MimeMessageHelper(mimeMessage, true, "UTF-8");
            message.setSubject(email.getTitle());
            message.setText(email.getContent(), true);
            message.setFrom(email.getSender());
            message.setTo(email.getRecipients().stream().map(s -> {
                try {
                    return new InternetAddress(s.getEmail());
                } catch (AddressException e) {
                    log.error("address is not valid {}", e.getRef(), e);
                    return null;
                }
            }).filter(Objects::nonNull).collect(Collectors.toList()).toArray(new InternetAddress[]{}));
            message.addInline(this.properties.getEmail().getLogoName(), getLogoFile(email.getKey()));
            message.addInline(this.properties.getEmail().getBackgroundImageName(), getBackgroundImageFile(email.getKey()));
        };
    }
```

근데 간혹 인라인으로 첨부한 이미지 파일들이 첨부파일로 들어가서 아래와 같이 noname으로 보일 때가 있다.  
<img width="1000" src="https://raw.githubusercontent.com/dlxotn216/image/master/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-02-28%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.13.07.png" />  
(Naver에선 그런 현상이 없었으나 Gmail에서 발생)

결론으론 addInline을 할 때 전달하는 FileResource의 filename으로부터 MimeType을 추출할 수 없는 상황이면 그런 듯 하다  
아래와 같이 FileResource에 확장자를 추가해주어서 해결했다.
<img width="1000" src="https://raw.githubusercontent.com/dlxotn216/image/master/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-02-28%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.18.20.png" />  
  
<img width="1000" src="https://raw.githubusercontent.com/dlxotn216/image/master/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202020-02-28%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.13.41.png" />
