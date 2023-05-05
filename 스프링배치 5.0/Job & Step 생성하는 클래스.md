### Job & Step 생성하는 클래스

자세한 내용 : [https://github.com/spring-projects/spring-batch/issues/4188](https://github.com/spring-projects/spring-batch/issues/4188)

**기존**

```
@EnableBatchProcessing
public class MyJobConfig {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Bean
    public Job job(Step step) {
        return this.jobBuilderFactory.get("myJob")
                .start(step)
                .build();
    }

}
```

[##_Image|kage@kPDE3/btsdZZ16brI/xRxciuiIrxQhhLMNb8aGbK/img.png|CDM|1.3|{"originWidth":700,"originHeight":197,"style":"alignCenter"}_##]

  
변경

```
@EnableBatchProcessing
public class MyJobConfig {

    @Bean
    public Job job(JobRepository jobRepository, Step step) {
        return new JobBuilder("myJob")
                .repository(jobRepository)
                .start(step)
                .build();
    }

}
```