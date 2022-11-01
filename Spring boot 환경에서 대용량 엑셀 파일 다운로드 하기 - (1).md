## 개요

게임 서버 개발 포지션에 있지만, 최근 게임과 관련된 API 서버를 인수인계 받으며 Spring boot + JPA 환경을 다룰일도 생겼다.

사내 게임의 FGT()를 진행하는데, 기획자 분이 매번 설문조사 결과를 요청하는 일이 있었다.

사실 설문조사 결과를 DB에 들어가 SQL문으로 뽑는 것은 살짝 귀찮고, 어려운 일은 전혀 아니다.

그러나, 굳이 불편하지 않는 일을 불편하다 생각해 무언가를 만드는게 개발자 특이니, API를 만들기로

한다.

## 요구사항

`spring boot`와 JPA 환경에서 DB에 있는 필요한 테이블에서 특정 조건(SQL)의 데이터를 csv 파일로 추출하는 API를 만든다.

예시로 2022년 1월부터 5월까지 게임에 결제한 유저 정보를 엑셀 파일로 추출하는 과정을 진행해본다.

### Entity

결제 정보 `Entity`이다. 여러 채널이 있는 게임이고, `PK`는 `user_no`로 구별한다.

```
@Getter @Setter @ToString  
@NoArgsConstructor  // 파라미터가 없는 기본 생성자를 생성하는 어노테이션
@AllArgsConstructor // 모든 필드 값을 파라미터로 받는 생성자 어노테이션 
@Builder  
@Entity
public class PayInfo {
    @Id  
    @Column(name = "user_no")  
    private String userNo;
    @Column(name = "channel_name")
    private String channelName;  
    @Column(name = "nickname")  
    private String nickName;
    @Column(name = "money")  
    private Long money;
    @Column(name = "pay_date")  
    private LocalDate PayDate;  
}
```

### Repository

`JPARepository`의 메소드를 사용하여, 원하는 데이터를 추출하는 메서드를 만들었다.

```
public interface PokerEventRepository extends JpaRepository<PayInfo, Long> {

List<PayInfo> findAllByChannelNamePayDateGreaterThanEqualAndPayDateLessThanEqual
(String channelName,LocalDate start, LocalDate end); // start 날짜부터 end 날짜 사이에 해당 채널에 있는 결제 정보 추출 
}
```

### Service

`Service` 전에 `control` 단계에서 받아올 `request class`를 하나 만들어준다.

```
@Getter @Setter @ToString  
@NoArgsConstructor  
public class DateRangeRequest {  
   private String channelName;
   @DateTimeFormat(pattern = "yyyy-MM-dd")  
   @JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd")  
   protected LocalDate from;  
   @JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd")  
   protected LocalDate to;  
}
```

JPA 메서드를 통해 DB에서 가져온 데이터들을 엑셀 파일 형식으로 바꿔준다.

  
`Workbook` 라이브러리를 사용하여, 엑셀 파일로 만드는 가장 basic한 형태이다.

코드를 보면 엑셀파일이 코드로 만들어지는지 감 잡을 수 있다.

또한 중요한 점은 **Return 타입으로 `ByteArrayInputStream`을 사용**한다는 것이다.

만약 Workbook의 객체를 그대로 사용했다면 body에서 무거운 workbook이 이동하는 것임으로 성능에 매우 좋지 않은 영향을 끼친다.

그러므로 **`ByteArrayInputStream`을 사용하여 데이터를 이동**시킨다.

또한, 엑셀 파일은 형식 자체의 압축기능이 들어가 있어 용량이 작아보이는 것이기 때문에 Cell 단위를 읽는 방식을 사용했다.

```
/**  
 * 엑셀파일 추출  
 *  
 * @param request  
 * @return  
 */  
public ByteArrayInputStream downloadExcelSurveyResult(DateRangeRequest request) {  
    String[] columns = {"유저 아이디", "채널이름", "닉네임", "결제머니"};  
    List<PokerEvent> payResults = eventRepository.findAllByChannelNamePayDateGreaterThanEqualAndPayDateLessThanEqual(  request.getChannelName(),request.getFrom(),request.getTo());  
    try (  
            Workbook workbook = new XSSFWorkbook();  
            ByteArrayOutputStream baos = new ByteArrayOutputStream();  
    ) {  
        Sheet sheet = workbook.createSheet(request.getFrom().toString() + " ~ ...");  
        int rowIndex = 0;  
        Row headerRow = sheet.createRow(rowIndex++);  
        for (int col = 0; col < columns.length; col++) {  
            Cell cell = headerRow.createCell(col);  
            cell.setCellValue(columns[col]);  
        }  

        for (PayInfo payResult : payResults) {  
        Row row = sheet.createRow(rowIndex++);  
        row.createCell(0).setCellValue(payResult.getUserId());  
        row.createCell(1).setCellValue(payResult.getChannelName());
        row.createCell(2).setCellValue(payResult.getNickName());
        row.createCell(3).setCellValue(payResult.getMoney());
        }  
        workbook.write(baos);  
        return new ByteArrayInputStream(baos.toByteArray());  
    } catch (Exception e) {  
        log.error("Fail to create excel file.");  
    }  
}
```

### Control

@ResponseBody 대신 `ResponseEntity` 통해서 엑셀파일을 보내는 이유는 FileDownload를 제공하는 응답을 만들어 내기 쉽기 때문에 사용한다.

이 두개의 차이점은 [@ResponseEntity가 뭘까? @ResponseBody와 차이점?](%5Bhttps://dev-coco.tistory.com/99%5D(https://dev-coco.tistory.com/99))에 잘 나와 있다.

```
@ApiOperation(value = "결제 내역 결과 엑셀 받기", notes = "결제 내역 결과를 엑셀 파일로 다운로드 받는다.")  

@ApiResponses({@ApiResponse(code = 200, message = "[header.status]<br/>&nbsp;&nbsp;" + "0 : 성공<br/>&nbsp;&nbsp;")})

@GetMapping(path = "payInfo/download-excel")
public ResponseEntity<InputStreamResource> downloadExcelPayResult(@ModelAttribute DateRangeRequest request) {  
    LocalDate today = LocalDate.now(); //생성될 엑셀 파일의 이름을 오늘날짜로 하기 위한 변수
    return ResponseEntity.ok()  // Status -> Header -> Body 순으로 작성 .headers(getExcelHeader("Pay_Result_"+today.getMonthValue()+"/"+today.getDayOfMonth()+".xlsx")).body(new InputStreamResource(pokerTestEventService.downloadExcelSurveyResult(request)));  
}


/**  
 * 엑셀 다운로드 헤더 생성  
 * @param fileName  
 * @return  
 */  
private HttpHeaders getExcelHeader(String fileName) {  
    HttpHeaders header = new HttpHeaders();  
    header.setContentType(  
            new MediaType("application", "vnd.openxmlformats-officedocument.spreadsheetml.sheet")  
    );  
    header.set(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=" + fileName);  
    return header;  
}
```

### 옵션

-   `vnd.openxmlformats-officedocument.spreadsheetml.shee` : .xlsx 확장자 다운로드를 위한 MediaType 옵션
-   `CONTENT_DISPOSITION`는 웹페이지에서 HTTP 프로토콜이 응답하는 데이터를 어떻게 표시하는지를 알려주는 Header이다. 기본값은 inline으로 설정되어 있으며, 이는 웹페이지에 표시하라는 의미이다. `attachment`를 사용한다면 사용자의 로컬에 다운로드 할 수 있도록 한다.

## 출처

Spring File Download 정리 - [https://move02.github.io/articles/2020-07/Spring-File-Download스프링부트](https://move02.github.io/articles/2020-07/Spring-File-Download%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8) REST API Excel download 방법 - [https://wildeveloperetrain.tistory.com/71](https://wildeveloperetrain.tistory.com/71)  
슬기로운 개발 생활 - : [https://dev-coco.tistory.com/99](https://dev-coco.tistory.com/99)