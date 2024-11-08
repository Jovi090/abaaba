import java.time.format.DateTimeParseException;

// 在 TradeController 类中
private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

public void recordNewTrade() {
    LocalDateTime tradedDatetime = null;
    while (tradedDatetime == null) {
        try {
            System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm): ");
            String datetimeInput = scanner.nextLine().trim();
            tradedDatetime = LocalDateTime.parse(datetimeInput, DATETIME_FORMATTER);
            
            // 额外检查日期有效性
            if (tradedDatetime.getDayOfMonth() > tradedDatetime.toLocalDate().lengthOfMonth()) {
                throw new DateTimeParseException("无效日期", datetimeInput, 0);
            }
            
        } catch (DateTimeParseException e) {
            System.out.println("無効な日時です。正しい形式で再入力してください。");
        }
    }

    System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
    String ticker = scanner.nextLine().trim();

    if (tradeRepository.isTickerRegistered(ticker)) {
        System.out.println("既に登録されている銘柄コードです。再度入力してください。");
        return;
    }

    System.out.print("銘柄名を入力してください: ");
    String tickerName = scanner.nextLine().trim();

    System.out.print("売買区分を入力してください (Buy/Sell): ");
    String side = scanner.nextLine().trim();

    System.out.print("数量を入力してください (100株単位): ");
    int quantity = Integer.parseInt(scanner.nextLine().trim());

    System.out.print("取引単価を入力してください (小数点以下2桁まで): ");
    BigDecimal tradedUnitPrice = new BigDecimal(scanner.nextLine().trim());

    LocalDateTime inputDatetime = LocalDateTime.now();

    Trade trade = new Trade(tradedDatetime, ticker, tickerName, side, quantity, tradedUnitPrice, inputDatetime);

    if (tradeRepository.saveTrade(trade)) {
        tradeView.showTradeAddedMessage(trade);
    } else {
        System.out.print("データの書き込みにエラーが発生しました。");
    }
}
