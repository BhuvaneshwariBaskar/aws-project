package com.trade.matching;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.*;

import java.util.*;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.stream.Collectors;

@SpringBootApplication
@EnableAsync
public class TradeMatchingApplication {
    public static void main(String[] args) {
        SpringApplication.run(TradeMatchingApplication.class, args);
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class Trade {
    private String tradeId;
    private String client;
    private String counterparty;
    private double amount;
    private int quantity;
    private String cusip;
    private String type; // BUY or SELL
}

@RestController
@RequestMapping("/trades")
class TradeController {
    @Autowired
    private TradeMatchingService tradeMatchingService;

    @PostMapping("/match")
    public Map<String, List<Trade>> matchTrades(@RequestBody List<Trade> trades) {
        return tradeMatchingService.matchTrades(trades);
    }
}

@Service
class TradeMatchingService {
    private final ExecutorService executorService = Executors.newFixedThreadPool(10);

    @Async
    public CompletableFuture<Map<String, List<Trade>>> matchTradesAsync(List<Trade> trades) {
        return CompletableFuture.supplyAsync(() -> matchTrades(trades), executorService);
    }

    public Map<String, List<Trade>> matchTrades(List<Trade> trades) {
        List<Trade> matched = new ArrayList<>();
        List<Trade> unmatched = new ArrayList<>();
        List<Trade> matchedWithError = new ArrayList<>();

        Map<String, List<Trade>> tradesById = trades.stream()
                .collect(Collectors.groupingBy(Trade::getTradeId));

        tradesById.forEach((tradeId, tradeList) -> {
            if (tradeList.size() == 2) {
                Trade trade1 = tradeList.get(0);
                Trade trade2 = tradeList.get(1);
                
                if (!trade1.getType().equals(trade2.getType()) &&
                        trade1.getClient().equals(trade2.getCounterparty()) &&
                        trade1.getCounterparty().equals(trade2.getClient()) &&
                        trade1.getAmount() == trade2.getAmount() &&
                        trade1.getQuantity() == trade2.getQuantity() &&
                        trade1.getCusip().equals(trade2.getCusip())) {
                    matched.add(trade1);
                    matched.add(trade2);
                } else {
                    matchedWithError.add(trade1);
                    matchedWithError.add(trade2);
                }
            } else {
                unmatched.addAll(tradeList);
            }
        });

        return Map.of(
                "Matched", matched,
                "Unmatched", unmatched,
                "MatchedWithError", matchedWithError
        );
    }
}
