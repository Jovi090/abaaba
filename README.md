package simplex.bn25.zhao335952.trading;

import simplex.bn25.zhao335952.trading.controller.MainController;
import simplex.bn25.zhao335952.trading.controller.StockController;
import simplex.bn25.zhao335952.trading.controller.TradeController;
import simplex.bn25.zhao335952.trading.model.repository.StockRepository;
import simplex.bn25.zhao335952.trading.model.repository.TradeRepository;
import simplex.bn25.zhao335952.trading.view.MenuView;
import simplex.bn25.zhao335952.trading.view.StockView;
import simplex.bn25.zhao335952.trading.view.TradeView;

public class Main {
    public static void main(String[] args) {
        MenuView menuView = new MenuView();
        StockView stockView = new StockView();
        TradeView tradeView = new TradeView();

        String stockCsvFilePath = "C:/Users/qqq/IdeaProjects/untitled/New code/src/csvファイル/Stock.csv";
        String tradeCsvFilePath = "C:/Users/qqq/IdeaProjects/untitled/New code/src/csvファイル/Trade.csv";

        StockRepository stockRepository = new StockRepository(stockCsvFilePath);
        TradeRepository tradeRepository = new TradeRepository(tradeCsvFilePath);

        StockController stockController = new StockController(stockRepository, stockView);
        TradeController tradeController = new TradeController(tradeRepository, tradeView);

        MainController mainController = new MainController(menuView, stockController, tradeController);
        mainController.start();
    }
}

package simplex.bn25.zhao335952.trading.controller;

import simplex.bn25.zhao335952.trading.view.MenuView;

public class MainController {
    private final MenuView menuView;
    private final StockController stockController;
    private final TradeController tradeController;

    public MainController(MenuView menuView, StockController stockController, TradeController tradeController) {
        this.menuView = menuView;
        this.stockController = stockController;
        this.tradeController = tradeController;
    }

    public void start() {
        boolean running = true;
        while (running) {
            String choice = menuView.displayMainMenu();
            switch (choice) {
                case "1" ->{
                    System.out.println("「銘柄マスタ一覧表示」が選択されました。");
                    stockController.displayAllStocks();}
                case "2" ->{
                    System.out.println("「銘柄マスタ新規登録」が選択されました。");
                    stockController.addNewStock();}
                case "3" ->{
                    System.out.println("「取引新規登録」が選択されました。");
                    tradeController.recordNewTrade();}
                case "4" ->{
                    System.out.println("「取引一覧表示」が選択されました。");
                    tradeController.displayAllTrades();}
                case "9" -> {
                    System.out.println("アプリケーションを終了します。");
                    running = false;
                }
                default ->  System.out.println("対応するメニューは存在しません。");
            }
            System.out.println("---");
        }
        menuView.close();
    }
}

package simplex.bn25.zhao335952.trading.controller;

import simplex.bn25.zhao335952.trading.model.Market;
import simplex.bn25.zhao335952.trading.model.Stock;
import simplex.bn25.zhao335952.trading.model.repository.StockRepository;
import simplex.bn25.zhao335952.trading.view.StockView;

import java.util.List;
import java.util.Scanner;

public class StockController {
    private final StockRepository stockRepository;
    private final StockView stockView;
    private final Scanner scanner = new Scanner(System.in);

    public StockController(StockRepository stockRepository, StockView stockView) {
        this.stockRepository = stockRepository;
        this.stockView = stockView;
    }

    public void displayAllStocks() {
        List<Stock> stocks = stockRepository.getAllStocks();
        stockView.displayStockList(stocks);
    }

    public void addNewStock() {
        String ticker = inputTicker();
        String productName = inputProductName();
        String market = inputMarket();
        long sharesIssued = inputSharesIssued();

        Stock stock = new Stock(ticker, productName, market, sharesIssued);

        if (stockRepository.saveStock(stock)) {
            stockView.showStockAddedMessage(stock);
        } else {
            System.out.print("データの書き込みにエラーが発生しました。");
        }
    }

    private String inputTicker() {
        String ticker;
        while (true) {
            System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
            ticker = scanner.nextLine().trim();

            if (!Stock.isValidTicker(ticker)) {
                System.out.println("無効な銘柄コードです。再度入力してください。");
                continue;
            }

            if (stockRepository.isTickerRegistered(ticker)) {
                System.out.println("既に登録されている銘柄コードです。再度入力してください。");
            } else {
                break;
            }
        }
        return ticker;
    }

    private String inputProductName() {
        String productName;
        while (true) {
            System.out.print("銘柄名を入力してください: ");
            productName = scanner.nextLine().trim();
            if (Stock.isValidProductName(productName)) {
                break;
            } else {
                System.out.println("無効な銘柄名です。再度入力してください。");
            }
        }
        return productName;
    }

    private String inputMarket() {
        String market;
        while (true) {
            System.out.print("上場市場を入力してください（Prime, Standard, Growth）：");
            market = scanner.nextLine().trim();
            if (Market.isValidMarket(market)) {
                break;
            } else {
                System.out.println("無効な上場市場です。再度入力してください。");
            }
        }
        return market;
    }

    private long inputSharesIssued() {
        long sharesIssued;
        while (true) {
            System.out.print("発行済み株式数を入力してください: ");
            String input = scanner.nextLine().trim();
            if (Stock.isValidSharesIssued(input)) {
                sharesIssued = Long.parseLong(input);
                break;
            } else {
                System.out.println("無効な株式発行数です。1から999999999999の間の整数を入力してください。");
            }
        }
        return sharesIssued;
    }
}
package simplex.bn25.zhao335952.trading.controller;

import simplex.bn25.zhao335952.trading.model.Trade;
import simplex.bn25.zhao335952.trading.model.repository.TradeRepository;
import simplex.bn25.zhao335952.trading.view.TradeView;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;
import java.util.Scanner;

public class TradeController {
    private final TradeRepository tradeRepository;
    private final TradeView tradeView;
    private final Scanner scanner = new Scanner(System.in);
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public TradeController(TradeRepository tradeRepository, TradeView tradeView) {
        this.tradeRepository = tradeRepository;
        this.tradeView = tradeView;
    }

    public void displayAllTrades() {
        List<Trade> trades = tradeRepository.getAllTrades();
        tradeView.displayTradeList(trades);
    }

    public void recordNewTrade() {
        System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm):");
        LocalDateTime tradedDatetime = LocalDateTime.parse(scanner.nextLine().trim(), DATETIME_FORMATTER);

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

        System.out.print("数量を入力してください (100株単位):");
        int quantity = Integer.parseInt(scanner.nextLine().trim());

        System.out.print("取引単価を入力してください (小数点以下2桁まで):");
        BigDecimal tradedUnitPrice = new BigDecimal(scanner.nextLine().trim());

        LocalDateTime inputDatetime = LocalDateTime.now();

        Trade trade = new Trade(tradedDatetime, ticker, tickerName, side, quantity, tradedUnitPrice, inputDatetime);

        if (tradeRepository.saveTrade(trade)) {
            tradeView.showTradeAddedMessage(trade);
        } else {
            System.out.print("データの書き込みにエラーが発生しました。");
        }
    }
}
package simplex.bn25.zhao335952.trading.model.repository;

import simplex.bn25.zhao335952.trading.model.Stock;
import simplex.bn25.zhao335952.trading.model.Market;
import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class StockRepository {
    private final String csvFilePath;

    public StockRepository(String csvFilePath) {
        this.csvFilePath = csvFilePath;
    }

    public List<Stock> getAllStocks() {
        List<Stock> stocks = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            br.readLine();
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length == 4) {
                    String ticker = fields[0].trim();
                    String productName = fields[1].trim();
                    String marketString = fields[2].trim();
                    long sharesIssued = Long.parseLong(fields[3].trim());

                    Market market = Market.fromString(marketString);
                    Stock stock = new Stock(ticker, productName, market.getSymbol(), sharesIssued);
                    stocks.add(stock);
                }
            }
        } catch (IOException e) {
            System.out.println("データの読み込み中にエラーが発生しました。");
        }
        return stocks;
    }

    public boolean saveStock(Stock stock) {
        try (FileWriter writer = new FileWriter(csvFilePath, true)) {
            writer.write(String.format("\n%s,%s,%s,%d",
                    stock.getTicker(),
                    stock.getProductName(),
                    stock.getMarket(),
                    stock.getSharesIssued()));
            return true;
        } catch (IOException e) {
            System.out.println("データの書き込みにエラーが発生しました。");
            return false;
        }
    }

    public boolean isTickerRegistered(String ticker) {
        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields[0].trim().equalsIgnoreCase(ticker)) {
                    return true;
                }
            }
        } catch (IOException e) {
            System.out.println("データが見つかりません。新しいCSVファイルを作成します。");
        }
        return false;
    }
}
package simplex.bn25.zhao335952.trading.model.repository;

import simplex.bn25.zhao335952.trading.model.Trade;

import java.io.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;

public class TradeRepository {
    private final String csvFilePath;
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public TradeRepository(String csvFilePath) {
        this.csvFilePath = csvFilePath;
    }

    public List<Trade> getAllTrades() {
        List<Trade> trades = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            br.readLine();
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length == 7) {
                    LocalDateTime tradedDatetime = LocalDateTime.parse(fields[0].trim(), DATETIME_FORMATTER);
                    String ticker = fields[1].trim();
                    String tickerName = fields[2].trim();
                    String side = fields[3].trim();
                    int quantity = Integer.parseInt(fields[4].trim());
                    BigDecimal tradedUnitPrice = new BigDecimal(fields[5].trim());
                    LocalDateTime inputDatetime = LocalDateTime.parse(fields[6].trim(), DATETIME_FORMATTER);

                    Trade trade = new Trade(tradedDatetime, ticker, tickerName, side, quantity, tradedUnitPrice, inputDatetime);
                    trades.add(trade);
                }
            }
        } catch (IOException e) {
            System.out.println("データの読み込み中にエラーが発生しました。");
        }

        return trades;
    }

    public boolean saveTrade(Trade trade) {
        try (FileWriter writer = new FileWriter(csvFilePath, true)) {
            writer.write("\n" + trade.toCSVFormat());
            return true;
        } catch (IOException e) {
            System.out.println("データの書き込み中にエラーが発生しました。");
            return false;
        }
    }

    public boolean isTickerRegistered(String ticker) {
        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length > 1 && fields[1].trim().equalsIgnoreCase(ticker)) {
                    return true;
                }
            }
        } catch (IOException e) {
            System.out.println("データの読み込み中にエラーが発生しました。");
        }
        return false;
    }

}
package simplex.bn25.zhao335952.trading.model;

public enum Market {
    PRIME("P", "Prime"),
    STANDARD("S", "Standard"),
    GROWTH("G", "Growth");

    private final String symbol;
    private final String fullName;

    Market(String symbol, String fullName) {
        this.symbol = symbol;
        this.fullName = fullName;
    }

    public String getSymbol() {
        return symbol;
    }

    public String getFullName() {
        return fullName;
    }

    public static Market fromString(String market) {
        String normalizedMarket = market.trim().toLowerCase();
        return switch (normalizedMarket) {
            case "prime", "プライム", "p" -> PRIME;
            case "standard", "スタンダード", "s" -> STANDARD;
            case "growth", "グロース", "g" -> GROWTH;
            default -> throw new IllegalArgumentException("無効な上場市場です。");
        };
    }

    public static boolean isValidMarket(String market) {
        try {
            fromString(market);
            return true;
        } catch (IllegalArgumentException e) {
            return false;
        }
    }
}
package simplex.bn25.zhao335952.trading.model;

public class Stock {
    private String ticker;
    private String productName;
    private Market market;
    private long sharesIssued;

    public Stock(String ticker, String productName, String market, long sharesIssued) {
        this.ticker = ticker;
        this.productName = productName;
        setMarket(market);
        this.sharesIssued = sharesIssued;
    }

    public String getTicker() {
        return ticker;
    }

    public void setTicker(String ticker) {
        if (isValidTicker(ticker)) {
            this.ticker = ticker;
        } else {
            throw new IllegalArgumentException("無効な銘柄コードです。");
        }
    }

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        if (isValidProductName(productName)) {
            this.productName = productName;
        } else {
            throw new IllegalArgumentException("無効な銘柄名です。");
        }
    }

    public String getMarket() {
        return market.getSymbol();
    }

    public void setMarket(String market) {
        this.market = Market.fromString(market);
    }

    public long getSharesIssued() {
        return sharesIssued;
    }

    public void setSharesIssued(long sharesIssued) {
        if (sharesIssued >= 1 && sharesIssued <= 999999999999L) {
            this.sharesIssued = sharesIssued;
        } else {
            throw new IllegalArgumentException("無効な株式発行数です。1から999999999999の間の整数を入力してください。");
        }
    }

    public static boolean isValidTicker(String ticker) {
        return ticker.matches("^[0-9][ACDFGHJKLMNPRSTUWXYacdfghjklmnprstuwxy0-9][0-9][ACDFGHJKLMNPRSTUWXYacdfghjklmnprstuwxy0-9]$");
    }

    public static boolean isValidProductName(String productName) {
        return productName.matches("^[A-Za-z0-9 .()]+$");
    }

    public static boolean isValidMarket(String market) {
        String normalizedMarket = market.trim().toLowerCase();
        return normalizedMarket.equals("prime") ||
                normalizedMarket.equals("プライム") ||
                normalizedMarket.equals("standard") ||
                normalizedMarket.equals("スタンダード") ||
                normalizedMarket.equals("growth") ||
                normalizedMarket.equals("グロース");
    }

    public static boolean isValidSharesIssued(String input) {
        try {
            long sharesIssued = Long.parseLong(input);
            return sharesIssued >= 1 && sharesIssued <= 999999999999L;
        } catch (NumberFormatException e) {
            return false;
        }
    }

    public String getMarketFullName() {
        return market.getFullName();
    }
}
package simplex.bn25.zhao335952.trading.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Trade {
    private LocalDateTime tradedDatetime;
    private String ticker;
    private String tickerName;
    private String side;
    private int quantity;
    private BigDecimal tradedUnitPrice;
    private LocalDateTime inputDatetime;

    public static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public Trade(LocalDateTime tradedDatetime, String ticker, String tickerName, String side, int quantity, BigDecimal tradedUnitPrice, LocalDateTime inputDatetime) {
        this.tradedDatetime = tradedDatetime;
        this.ticker = ticker;
        this.tickerName = tickerName;
        setSide(side);
        setQuantity(quantity);
        this.tradedUnitPrice = tradedUnitPrice;
        this.inputDatetime = inputDatetime;
    }

    public LocalDateTime getTradedDatetime() {
        return tradedDatetime;
    }

    public void setTradedDatetime(LocalDateTime tradedDatetime) {
        this.tradedDatetime = tradedDatetime;
    }

    public String getTicker() {
        return ticker;
    }

    public void setTicker(String ticker) {
        this.ticker = ticker;
    }

    public String getTickerName() {
        return tickerName;
    }

    public void setTickerName(String tickerName) {
        this.tickerName = tickerName;
    }

    public String getSide() {
        return side;
    }

    public void setSide(String side) {
        if (isValidSide(side)) {
            this.side = side;
        } else {
            throw new IllegalArgumentException("BuyまたはSellを入力してください。");
        }
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        if (quantity > 0 && quantity % 100 == 0) { // 假设每笔交易为100股的倍数
            this.quantity = quantity;
        } else {
            throw new IllegalArgumentException("100の倍数である数値を入力してください。");
        }
    }

    public BigDecimal getTradedUnitPrice() {
        return tradedUnitPrice;
    }

    public void setTradedUnitPrice(BigDecimal tradedUnitPrice) {
        this.tradedUnitPrice = tradedUnitPrice;
    }

    public LocalDateTime getInputDatetime() {
        return inputDatetime;
    }

    public void setInputDatetime(LocalDateTime inputDatetime) {
        this.inputDatetime = inputDatetime;
    }

    private boolean isValidSide(String side) {
        return "Buy".equalsIgnoreCase(side) || "Sell".equalsIgnoreCase(side);
    }

    public String toCSVFormat() {
        return String.join(",",
                tradedDatetime.format(DATETIME_FORMATTER),
                ticker,
                tickerName,
                side,
                String.valueOf(quantity),
                tradedUnitPrice.toString(),
                inputDatetime.format(DATETIME_FORMATTER)
        );
    }
}
package simplex.bn25.zhao335952.trading.view;

import java.util.Scanner;

public class MenuView {
    private final Scanner scanner;

    public MenuView() {
        scanner = new Scanner(System.in);
    }

    public String displayMainMenu() {
        System.out.println("操作するメニューを選んでください。");
        System.out.println("1. 銘柄マスタ一覧表示");
        System.out.println("2. 銘柄マスタ新規登録");
        System.out.println("3. 取引新規登録");
        System.out.println("4. 取引一覧表示");
        System.out.println("9. アプリケーションを終了する");
        System.out.print("入力してください: ");

        return scanner.nextLine().trim();
    }

    public void close() {
        scanner.close();
    }

}
package simplex.bn25.zhao335952.trading.view;

import simplex.bn25.zhao335952.trading.model.Stock;
import java.util.List;

public class StockView {

    public void displayStockList(List<Stock> stocks) {
        if (stocks.isEmpty()) {
            System.out.println("データが見つかりません。CSVファイルを確認してください。");
        } else {
            System.out.println("------------------------------------------------------------------------");
            System.out.printf("| %-6s | %-30s | %-8s | %15s |%n", "Ticker", "Product Name", "Market", "Shares Issued");
            System.out.println("------------------------------------------------------------------------");
            for (Stock stock : stocks) {
                displayStock(stock);
            }
            System.out.println("------------------------------------------------------------------------");
        }
    }

    public void displayStock(Stock stock) {
        String productName = stock.getProductName();
        if (productName.length() > 30) {
            productName = productName.substring(0, 27) + "...";
        }
        System.out.printf("| %-6s | %-30s | %-8s | %15s |%n",
                stock.getTicker().toUpperCase(),
                productName,
                stock.getMarketFullName(),
                formatSharesIssued(stock.getSharesIssued()));
    }

    public void showStockAddedMessage(Stock stock) {
        System.out.println("銘柄マスタ新規登録しました:" + stock.getProductName());
    }

    private String formatSharesIssued(long sharesIssued) {
        return String.format("%,d", sharesIssued);
    }

}
package simplex.bn25.zhao335952.trading.view;

import simplex.bn25.zhao335952.trading.model.Trade;
import java.util.List;

public class TradeView {

    public void displayTradeList(List<Trade> trades) {
        if (trades.isEmpty()) {
            System.out.println("取引データが見つかりません。CSVファイルを確認してください。");
        } else {
            System.out.println("-------------------------------------------------------------------------------");
            System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10s | %10s |%n",
                    "Datetime","Ticker","Product Name","Side","Quantity","Unit Price");
            System.out.println("-------------------------------------------------------------------------------");
            for (Trade trade : trades) {
                displayTrade(trade);
            }
            System.out.println("-------------------------------------------------------------------------------");
        }
    }

    public void displayTrade(Trade trade) {
        System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10d | %10.2f |%n",
                trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
                trade.getTicker().toUpperCase(),
                trade.getTickerName(),
                trade.getSide(),
                trade.getQuantity(),
                trade.getTradedUnitPrice());
    }

    public void showTradeAddedMessage(Trade trade) {
        System.out.println("取引データを新規登録しました。" + trade.getTickerName() + " (" + trade.getTicker() + ")");
    }
}

