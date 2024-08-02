
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import io.github.resilience4j.ratelimiter.RateLimiter;
import io.github.resilience4j.ratelimiter.RateLimiterConfig;
import lombok.Builder;
import lombok.Data;
import lombok.extern.slf4j.Slf4j;

import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;
import java.time.LocalDate;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

@Slf4j
public class CrptApi {

    /**
     * В качестве лимитера использовал библиотеку resilience4j, можно конечно и на Semaphore написать;
     * Сериализацию делал через Jackson, использовал обычные классы, можно попробовать с Records, но на практике редко применял!
     * Основной метод сделал CompletableFuture.
     * */
    private final HttpClient httpClient;
    private final ObjectMapper objectMapper;
    private final RateLimiter rateLimiter;
    private final String url = "https://ismp.crpt.ru/api/v3/lk/documents/create";


    public CrptApi(TimeUnit timeUnit, int requestLimit) {
        this.httpClient = HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(10))
                .build();
        this.objectMapper = new ObjectMapper().registerModule(new JavaTimeModule());

        RateLimiterConfig config = RateLimiterConfig.custom()
                .limitRefreshPeriod(Duration.of(1, timeUnit.toChronoUnit()))
                .limitForPeriod(requestLimit)
                .timeoutDuration(Duration.ofSeconds(5))
                .build();
        this.rateLimiter = RateLimiter.of("crptApi", config);
    }

    public CompletableFuture<String> createDocument(Document document, String signature){
        return CompletableFuture.supplyAsync(() -> {
            try {

                if(!rateLimiter.acquirePermission()){
                    System.out.println("Лимит превышен");
                    //Тут можно что-то сделать, какой-нибудь паттерн Retry
                    return null;
                }
                String jsonDocument = objectMapper.writeValueAsString(document);
                HttpRequest request = HttpRequest.newBuilder()
                        .uri(URI.create(url))
                        .header("Content-Type", "application/json")
                        .header("Signature", signature)
                        .POST(HttpRequest.BodyPublishers.ofString(jsonDocument))
                        .build();

                HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
                log.info("Создание прошло успешно");
                return response.body();
            } catch (Exception e) {
                log.error("Ошибка создания", e);
                throw new RuntimeException("Error creating document", e);
            }
        });
    }

    @Data
    @Builder
    public static class Description {
        private String participantInn;
    }

    @Data
    @Builder
    public static class Product {
        private String certificateDocument;
        private LocalDate certificateDocumentDate;
        private String certificateDocumentNumber;
        private String ownerInn;
        private String producerInn;
        private LocalDate productionDate;
        private String tnvedCode;
        private String uitCode;
        private String uituCode;
    }

    @Data
    @Builder
    public static class Document {
        private Description description;
        private String docId;
        private String docStatus;
        private DocType docType;
        private boolean importRequest;
        private String ownerInn;
        private String participantInn;
        private String producerInn;
        private LocalDate productionDate;
        private String productionType;
        private List<Product> products;
        private LocalDate regDate;
        private String regNumber;
    }


    public enum DocType {
        LP_INTRODUCE_GOODS
    }

}
