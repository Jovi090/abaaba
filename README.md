public void displayTradeList(List<Trade> trades) {
    if (trades.isEmpty()) {
        System.out.println("取引データが見つかりません。CSVファイルを確認してください。");
    } else {
        System.out.println("-------------------------------------------------------------------------------");
        System.out.printf("| %-"+WIDTH_DATETIME+"s | %-"+WIDTH_TICKER+"s | %-"+WIDTH_PRODUCT_NAME+"s | %-"+WIDTH_SIDE+"s | %"+WIDTH_QUANTITY+"s | %"+WIDTH_UNIT_PRICE+"s |%n",
                "Datetime", "Ticker", "Product Name", "Side", "Quantity", "Unit Price");
        System.out.println("-------------------------------------------------------------------------------");
        for (Trade trade : trades) {
            displayTrade(trade);
        }
        System.out.println("-------------------------------------------------------------------------------");
    }
}

public void displayTrade(Trade trade) {
    System.out.printf("| %-"+WIDTH_DATETIME+"s | %-"+WIDTH_TICKER+"s | %-"+WIDTH_PRODUCT_NAME+"s | %-"+WIDTH_SIDE+"s | %"+WIDTH_QUANTITY+"d | %"+WIDTH_UNIT_PRICE+".2f |%n",
            trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
            trade.getTicker().toUpperCase(),
            trade.getTickerName(),
            trade.getSide(),
            trade.getQuantity(),
            trade.getTradedUnitPrice());
}

public class TradeView {

    private static final int WIDTH_DATETIME = 19;
    private static final int WIDTH_TICKER = 6;
    private static final int WIDTH_PRODUCT_NAME = 30;
    private static final int WIDTH_SIDE = 4;
    private static final int WIDTH_QUANTITY = 10;
    private static final int WIDTH_UNIT_PRICE = 10;

    // other methods...
