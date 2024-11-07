import java.text.DecimalFormat;

public class TradeView {

    private final StockRepository stockRepository;

    public TradeView(StockRepository stockRepository) {
        this.stockRepository = stockRepository;
    }

    public void displayTradeList(List<Trade> trades) {
        if (trades.isEmpty()) {
            System.out.println("取引データが見つかりません。CSVファイルを確認してください。");
        } else {
            System.out.println("-------------------------------------------------------------------------------");
            System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10s | %15s |%n",
                    "Datetime", "Ticker", "Product Name", "Side", "Quantity", "Unit Price");
            System.out.println("-------------------------------------------------------------------------------");
            for (Trade trade : trades) {
                displayTrade(trade);
            }
            System.out.println("-------------------------------------------------------------------------------");
        }
    }

    public void displayTrade(Trade trade) {
        // 使用 stockRepository 根据 ticker 获取对应的股票名称
        String productName = stockRepository.getProductNameByTicker(trade.getTicker());
        
        // 调用 formatUnitPrice 方法进行千分位格式化
        String formattedUnitPrice = formatUnitPrice(trade.getTradedUnitPrice());

        System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10d | %15s |%n",
                trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
                trade.getTicker().toUpperCase(),
                productName != null ? productName : "N/A",  // 如果找不到名称，显示 "N/A"
                trade.getSide().getValue(),
                trade.getQuantity(),
                formattedUnitPrice);  // 使用格式化后的单价
    }

    // 新增的方法：格式化单价为千分位格式
    private String formatUnitPrice(BigDecimal unitPrice) {
        DecimalFormat decimalFormat = new DecimalFormat("#,##0.00");
        return decimalFormat.format(unitPrice);
    }
}

public class TradeView {

    private final StockRepository stockRepository;

    public TradeView(StockRepository stockRepository) {
        this.stockRepository = stockRepository;
    }

    public void displayTradeList(List<Trade> trades) {
        if (trades.isEmpty()) {
            System.out.println("取引データが見つかりません。CSVファイルを確認してください。");
        } else {
            // 定义每列的宽度
            int datetimeWidth = 19;
            int tickerWidth = 6;
            int productNameWidth = 30;
            int sideWidth = 4;
            int quantityWidth = 10;
            int unitPriceWidth = 15;

            // 输出标题行
            System.out.println("-------------------------------------------------------------------------------");
            System.out.println(formatTradeRow("Datetime", "Ticker", "Product Name", "Side", "Quantity", "Unit Price",
                    datetimeWidth, tickerWidth, productNameWidth, sideWidth, quantityWidth, unitPriceWidth));
            System.out.println("-------------------------------------------------------------------------------");

            // 输出每条交易数据
            for (Trade trade : trades) {
                displayTrade(trade, datetimeWidth, tickerWidth, productNameWidth, sideWidth, quantityWidth, unitPriceWidth);
            }
            System.out.println("-------------------------------------------------------------------------------");
        }
    }

    public void displayTrade(Trade trade, int datetimeWidth, int tickerWidth, int productNameWidth,
                             int sideWidth, int quantityWidth, int unitPriceWidth) {
        String productName = stockRepository.getProductNameByTicker(trade.getTicker());
        String formattedUnitPrice = formatUnitPrice(trade.getTradedUnitPrice());

        System.out.println(formatTradeRow(
                trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
                trade.getTicker().toUpperCase(),
                productName != null ? productName : "N/A",
                trade.getSide().getValue(),
                String.valueOf(trade.getQuantity()),
                formattedUnitPrice,
                datetimeWidth, tickerWidth, productNameWidth, sideWidth, quantityWidth, unitPriceWidth));
    }

    // 根据列宽动态生成格式化字符串
    private String formatTradeRow(String datetime, String ticker, String productName, String side,
                                  String quantity, String unitPrice,
                                  int datetimeWidth, int tickerWidth, int productNameWidth,
                                  int sideWidth, int quantityWidth, int unitPriceWidth) {
        String formatString = String.format("| %%-%ds | %%-%ds | %%-%ds | %%-%ds | %%-%ds | %%-%ds |",
                datetimeWidth, tickerWidth, productNameWidth, sideWidth, quantityWidth, unitPriceWidth);
        return String.format(formatString, datetime, ticker, productName, side, quantity, unitPrice);
    }

    private String formatUnitPrice(BigDecimal unitPrice) {
        DecimalFormat decimalFormat = new DecimalFormat("#,##0.00");
        return decimalFormat.format(unitPrice);
    }
}
