import simplex.bn25.zhao335952.trading.model.repository.StockRepository;
import simplex.bn25.zhao335952.trading.model.Stock;
import simplex.bn25.zhao335952.trading.model.Trade;
import java.util.List;

public class TradeView {

    private final StockRepository stockRepository;

    // 在构造函数中添加 StockRepository
    public TradeView(StockRepository stockRepository) {
        this.stockRepository = stockRepository;
    }

    public void displayTradeList(List<Trade> trades) {
        if (trades.isEmpty()) {
            System.out.println("取引データが見つかりません。CSVファイルを確認してください。");
        } else {
            System.out.println("-------------------------------------------------------------------------------");
            System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10s | %10s |%n",
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
        
        System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10d | %10.2f |%n",
                trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
                trade.getTicker().toUpperCase(),
                productName != null ? productName : "N/A",  // 如果找不到名称，显示 "N/A"
                trade.getSide().getValue(),
                trade.getQuantity(),
                trade.getTradedUnitPrice());
    }
}

sr
public String getProductNameByTicker(String ticker) {
    List<Stock> stocks = getAllStocks();  // 从 CSV 文件中获取所有股票
    for (Stock stock : stocks) {
        if (stock.getTicker().equalsIgnoreCase(ticker)) {
            return stock.getProductName();
        }
    }
    return null;  // 如果找不到对应的股票代码，返回 null
}

// Main.java

public class Main {
    public static void main(String[] args) {
        MenuView menuView = new MenuView();
        StockView stockView = new StockView();
        
        String stockCsvFilePath = "C:/path/to/Stock.csv";
        String tradeCsvFilePath = "C:/path/to/Trade.csv";

        StockRepository stockRepository = new StockRepository(stockCsvFilePath);  // 初始化 StockRepository
        TradeRepository tradeRepository = new TradeRepository(tradeCsvFilePath);

        StockController stockController = new StockController(stockRepository, stockView);
        TradeView tradeView = new TradeView(stockRepository);  // 将 StockRepository 传递给 TradeView
        TradeController tradeController = new TradeController(tradeRepository, tradeView);

        MainController mainController = new MainController(menuView, stockController, tradeController);
        mainController.start();
    }
}

// TradeController.java

public class TradeController {
    private final TradeRepository tradeRepository;
    private final TradeView tradeView;

    public TradeController(TradeRepository tradeRepository, TradeView tradeView) {
        this.tradeRepository = tradeRepository;
        this.tradeView = tradeView;
    }

    public void displayAllTrades() {
        List<Trade> trades = tradeRepository.getAllTrades();

        // 按交易日期降序排序
        trades.sort(Comparator.comparing(Trade::getTradedDatetime).reversed());

        // 显示交易列表
        tradeView.displayTradeList(trades);
    }
}
