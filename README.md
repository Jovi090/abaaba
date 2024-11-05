package app;

import controller.MainController;
import controller.StockController;
import controller.TradeController;
import model.repository.StockRepository;
import model.repository.TradeRepository;
import view.MenuView;
import view.StockView;
import view.TradeView;

public class Main {
    public static void main(String[] args) {
        // 1. 初始化 View 层
        MenuView menuView = new MenuView();
        StockView stockView = new StockView();
        TradeView tradeView = new TradeView();

        // 2. 初始化 Model 层的 Repository
        // 设置CSV文件路径
        String stockCsvFilePath = "C:/Users/qqq/IdeaProjects/untitled/New code/src/csvファイル/Stock.csv";
        String tradeCsvFilePath = "C:/Users/qqq/IdeaProjects/untitled/New code/src/csvファイル/Trade.csv";

        StockRepository stockRepository = new StockRepository(stockCsvFilePath);
        TradeRepository tradeRepository = new TradeRepository(tradeCsvFilePath);

        // 3. 初始化 Controller 层
        StockController stockController = new StockController(stockRepository, stockView);
        TradeController tradeController = new TradeController(tradeRepository, tradeView);

        // 4. 初始化 MainController 并启动应用程序
        MainController mainController = new MainController(menuView, stockController, tradeController);
        mainController.start();
    }
}

package controller;

import view.MenuView;

public class MainController {
    private final MenuView menuView;
    private final StockController stockController;
    private final TradeController tradeController;

    public MainController(MenuView menuView, StockController stockController, TradeController tradeController) {
        this.menuView = menuView;
        this.stockController = stockController;
        this.tradeController = tradeController;
    }

    // 主流程控制
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
        menuView.close();  // 关闭资源
    }
}

package controller;

import model.Stock;
import model.repository.StockRepository;
import view.StockView;

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

    // 显示所有股票
    public void displayAllStocks() {
        List<Stock> stocks = stockRepository.getAllStocks();
        stockView.displayStockList(stocks);
    }

    // 添加新股票
    public void addNewStock() {
        String ticker = inputTicker();
        String productName = inputProductName();
        String market = inputMarket();
        long sharesIssued = inputSharesIssued();

        // 创建股票对象
        Stock stock = new Stock(ticker, productName, market, sharesIssued);

        // 保存股票对象
        if (stockRepository.saveStock(stock)) {
            stockView.showStockAddedMessage(stock);
        } else {
            System.out.print("股票添加失败，请检查输入数据。");
        }
    }

    // 输入股票代码并检查唯一性
    private String inputTicker() {
        String ticker;
        while (true) {
            System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
            ticker = scanner.nextLine().trim();

            if (!Stock.isValidTicker(ticker)) {
                System.out.println("無効な銘柄コードです。再度入力してください。");
                continue;
            }

            // 在此处检查唯一性
            if (stockRepository.isTickerRegistered(ticker)) {
                System.out.println("既に登録されている銘柄コードです。再度入力してください。");
            } else {
                break; // 验证通过并且股票代码不存在，跳出循环
            }
        }
        return ticker;
    }

    // 输入股票名称
    private String inputProductName() {
        String productName;
        while (true) {
            System.out.print("銘柄名を入力してください: ");
            productName = scanner.nextLine().trim();
            if (Stock.isValidProductName(productName)) {
                break; // 如果验证通过，跳出循环
            } else {
                System.out.println("無効な銘柄名です。再度入力してください。");
            }
        }
        return productName;
    }

    // 输入市场类型
    private String inputMarket() {
        String market;
        while (true) {
            System.out.print("上場市場を入力してください（Prime, Standard, Growth）：");
            market = scanner.nextLine().trim();
            if (Stock.isValidMarket(market)) {
                break; // 如果验证通过，跳出循环
            } else {
                System.out.println("無効な上場市場です。再度入力してください。");
            }
        }
        return market;
    }

    // 输入已发行股票数量
    private long inputSharesIssued() {
        long sharesIssued;
        while (true) {
            System.out.print("発行済み株式数を入力してください: ");
            String input = scanner.nextLine().trim();
            if (Stock.isValidSharesIssued(input)) {
                sharesIssued = Long.parseLong(input); // 由于已验证，所以可以安全地解析
                break; // 验证通过后跳出循环
            } else {
                System.out.println("無効な株式発行数です。1から999999999999の間の整数を入力してください。");
            }
        }
        return sharesIssued;
    }
}

package controller;

import model.Trade;
import model.repository.TradeRepository;
import view.TradeView;

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

    // 显示所有交易记录
    public void displayAllTrades() {
        List<Trade> trades = tradeRepository.getAllTrades();
        tradeView.displayTradeList(trades);
    }

    // 记录新交易
    public void recordNewTrade() {
        System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm):");
        LocalDateTime tradedDatetime = LocalDateTime.parse(scanner.nextLine().trim(), DATETIME_FORMATTER);

        System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
        String ticker = scanner.nextLine().trim();

        if (tradeRepository.isTickerRegistered(ticker)) {
            System.out.println("既に登録されている銘柄コードです。再度入力してください。");
            return; // 结束方法，不再要求输入其他信息
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

package model;

public class Stock {
    private String ticker;         // 股票代码
    private String productName;    // 股票名称
    private String market;         // 市场类别，存储为 P, S, G
    private long sharesIssued;     // 已发行股票数量

    // 构造方法
    public Stock(String ticker, String productName, String market, long sharesIssued) {
        this.ticker = ticker;
        this.productName = productName;
        setMarket(market); // 使用 setter 方法进行验证和设置
        this.sharesIssued = sharesIssued;
    }

    // Getter 和 Setter 方法
    public String getTicker() {
        return ticker;
    }

    public void setTicker(String ticker) {
        if (isValidTicker(ticker)) {
            this.ticker = ticker;
        } else {
            throw new IllegalArgumentException("无效的股票代码");
        }
    }

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        if (isValidProductName(productName)) {
            this.productName = productName;
        } else {
            throw new IllegalArgumentException("无效的股票名称");
        }
    }

    public String getMarket() {
        return market;
    }

    public void setMarket(String market) {
        // 根据用户输入的市场类型设置内部代码
        String normalizedMarket = market.trim().toLowerCase(); // 转为小写并去除前后空格
        switch (normalizedMarket) {
            case "prime":
            case "プライム":
                this.market = "P"; // Prime
                break;
            case "standard":
            case "スタンダード":
                this.market = "S"; // Standard
                break;
            case "growth":
            case "グロース":
                this.market = "G"; // Growth
                break;
            case "p":
                this.market = "P"; // 允许用户输入"P"
                break;
            case "s":
                this.market = "S"; // 允许用户输入"S"
                break;
            case "g":
                this.market = "G"; // 允许用户输入"G"
                break;
            default:
                throw new IllegalArgumentException("无效的上场市场"); // 提示错误信息
        }
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

    // 辅助方法：验证股票代码格式
    public static boolean isValidTicker(String ticker) {
        return ticker.matches("^[0-9][ACDFGHJKLMNPRSTUWXYacdfghjklmnprstuwxy0-9][0-9][ACDFGHJKLMNPRSTUWXYacdfghjklmnprstuwxy0-9]$");
    }

    // 辅助方法：验证股票名称格式
    public static boolean isValidProductName(String productName) {
        return productName.matches("^[A-Za-z0-9 .()]+$");
    }

    // 辅助方法：验证市场类别
    public static boolean isValidMarket(String market) {
        String normalizedMarket = market.trim().toLowerCase(); // 转为小写并去除前后空格
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
            return false; // 解析失败，返回无效
        }
    }

    public String getMarketFullName() {
        return switch (market) {
            case "P" -> "Prime";
            case "S" -> "Standard";
            case "G" -> "Growth";
            default -> "Unknown";
        };
    }
}

package model;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Trade {
    private LocalDateTime tradedDatetime;  // 交易时间
    private String ticker;                 // 股票代码
    private String tickerName;             // 股票名称
    private String side;                   // 买卖方向（Buy 或 Sell）
    private int quantity;                  // 交易数量
    private BigDecimal tradedUnitPrice;    // 交易单价
    private LocalDateTime inputDatetime;   // 输入时间

    public static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    // 构造方法
    public Trade(LocalDateTime tradedDatetime, String ticker, String tickerName, String side, int quantity, BigDecimal tradedUnitPrice, LocalDateTime inputDatetime) {
        this.tradedDatetime = tradedDatetime;
        this.ticker = ticker;
        this.tickerName = tickerName;
        setSide(side);  // 使用setter方法来验证输入
        setQuantity(quantity);  // 使用setter方法来验证输入
        this.tradedUnitPrice = tradedUnitPrice;
        this.inputDatetime = inputDatetime;
    }

    // Getter 和 Setter 方法
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
            throw new IllegalArgumentException("无效的买卖方向，应为 'Buy' 或 'Sell'");
        }
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        if (quantity > 0 && quantity % 100 == 0) { // 假设每笔交易为100股的倍数
            this.quantity = quantity;
        } else {
            throw new IllegalArgumentException("数量必须为正且是100的倍数");
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

package model.repository;

import model.Stock;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

public class StockRepository {
    private final String csvFilePath;

    // 构造方法，接收CSV文件路径
    public StockRepository(String csvFilePath) {
        this.csvFilePath = csvFilePath;
    }

    // 从CSV文件中读取所有股票数据
    public List<Stock> getAllStocks() {
        List<Stock> stocks = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            br.readLine(); // 跳过标题行
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length == 4) {
                    String ticker = fields[0].trim();
                    String productName = fields[1].trim();
                    String market = fields[2].trim();
                    long sharesIssued = Long.parseLong(fields[3].trim());

                    Stock stock = new Stock(ticker, productName, market, sharesIssued);
                    stocks.add(stock);
                }
            }
        } catch (IOException e) {
            System.out.println("データの読み込み中にエラーが発生しました。");
        }

        return stocks;
    }

    // 将新股票保存到CSV文件
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

    // 检查股票代码是否已存在
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
            System.out.println("銘柄コードの読み込み中にエラーが発生しました。");
        }
        return false;
    }
}

package model.repository;

import model.Trade;

import java.io.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;

public class TradeRepository {
    private final String csvFilePath;
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    // 构造方法，接收CSV文件路径
    public TradeRepository(String csvFilePath) {
        this.csvFilePath = csvFilePath;
    }

    // 从CSV文件中读取所有交易记录
    public List<Trade> getAllTrades() {
        List<Trade> trades = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            br.readLine(); // 跳过标题行
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
            System.err.println("读取交易数据时发生错误：" + e.getMessage());
        }

        return trades;
    }

    // 将新的交易记录保存到CSV文件
    public boolean saveTrade(Trade trade) {
        try (FileWriter writer = new FileWriter(csvFilePath, true)) {
            writer.write("\n" + trade.toCSVFormat());
            return true;
        } catch (IOException e) {
            System.err.println("保存交易数据时发生错误：" + e.getMessage());
            return false;
        }
    }

    public boolean isTickerRegistered(String ticker) {
        try (BufferedReader br = new BufferedReader(new FileReader(csvFilePath))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length > 1 && fields[1].trim().equalsIgnoreCase(ticker)) {
                    return true; // 找到匹配的股票代码
                }
            }
        } catch (IOException e) {
            System.err.println("检查股票代码时发生错误：" + e.getMessage());
        }
        return false; // 股票代码未找到
    }

}

package view;

import java.util.Scanner;

public class MenuView {
    private final Scanner scanner;

    public MenuView() {
        scanner = new Scanner(System.in);
    }

    // 显示主菜单，并返回用户的选择
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

    // 关闭 Scanner 资源
    public void close() {
        scanner.close();
    }

}

package view;

import model.Stock;
import java.util.List;

public class StockView {

    // 显示所有股票列表
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

    // 显示单个股票的详细信息
    public void displayStock(Stock stock) {
        String productName = stock.getProductName();
        if (productName.length() > 30) {
            productName = productName.substring(0, 27) + "...";
        }
        System.out.printf("| %-6s | %-30s | %-8s | %15s |%n",
                stock.getTicker(),
                productName,
                stock.getMarketFullName(),
                formatSharesIssued(stock.getSharesIssued()));
    }

    // 显示股票新增成功的提示
    public void showStockAddedMessage(Stock stock) {
        System.out.println("銘柄マスタ新規登録しました:" + stock.getProductName());
    }

    private String formatSharesIssued(long sharesIssued) {
        return String.format("%,d", sharesIssued);
    }

}

package view;

import model.Trade;
import java.util.List;

public class TradeView {

    // 显示所有交易记录
    public void displayTradeList(List<Trade> trades) {
        if (trades.isEmpty()) {
            System.out.println("交易记录为空。");
        } else {
            System.out.println("-------------------------------------------------------------------------------");
            System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10s | %10s |%n",
                    "交易时间", "代码", "名称", "方向", "数量", "单价");
            System.out.println("-------------------------------------------------------------------------------");
            for (Trade trade : trades) {
                displayTrade(trade);
            }
            System.out.println("-------------------------------------------------------------------------------");
        }
    }

    // 显示单个交易的详细信息
    public void displayTrade(Trade trade) {
        System.out.printf("| %-19s | %-6s | %-30s | %-4s | %10d | %10.2f |%n",
                trade.getTradedDatetime().format(Trade.DATETIME_FORMATTER),
                trade.getTicker(),
                trade.getTickerName(),
                trade.getSide(),
                trade.getQuantity(),
                trade.getTradedUnitPrice());
    }

    // 显示交易新增成功的提示
    public void showTradeAddedMessage(Trade trade) {
        System.out.println("取引データを新規登録しました。" + trade.getTickerName() + " (" + trade.getTicker() + ")");
    }
}







package model;

import java.math.BigDecimal;
import java.time.DayOfWeek;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;

public class Trade {
    private LocalDateTime tradedDatetime;  // 交易时间
    private String ticker;                  // 股票代码
    private String tickerName;              // 股票名称
    private String side;                    // 买卖方向（Buy 或 Sell）
    private int quantity;                   // 交易数量
    private BigDecimal tradedUnitPrice;     // 交易单价
    private LocalDateTime inputDatetime;    // 输入时间

    public static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    // 构造方法
    public Trade(LocalDateTime tradedDatetime, String ticker, String tickerName, String side, int quantity, BigDecimal tradedUnitPrice, LocalDateTime inputDatetime) {
        validateTradedDatetime(tradedDatetime); // 验证交易时间
        this.tradedDatetime = tradedDatetime;
        this.ticker = ticker;
        this.tickerName = tickerName;
        setSide(side);  // 使用setter方法来验证输入
        setQuantity(quantity);  // 使用setter方法来验证输入
        this.tradedUnitPrice = tradedUnitPrice;
        this.inputDatetime = inputDatetime;
    }

    // Getter 和 Setter 方法
    public LocalDateTime getTradedDatetime() {
        return tradedDatetime;
    }

    public void setTradedDatetime(LocalDateTime tradedDatetime) {
        validateTradedDatetime(tradedDatetime); // 验证交易时间
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
            throw new IllegalArgumentException("无效的买卖方向，应为 'Buy' 或 'Sell'");
        }
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        if (quantity > 0 && quantity % 100 == 0) { // 假设每笔交易为100股的倍数
            this.quantity = quantity;
        } else {
            throw new IllegalArgumentException("数量必须为正且是100的倍数");
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

    private void validateTradedDatetime(LocalDateTime tradedDatetime) {
        // 当前时间
        LocalDateTime now = LocalDateTime.now();
        
        // 1. 检查是否为未来的时间
        if (tradedDatetime.isAfter(now)) {
            throw new IllegalArgumentException("交易时间不能是未来的时间。");
        }

        // 2. 检查年份是否大于1878年
        if (tradedDatetime.getYear() <= 1878) {
            throw new IllegalArgumentException("年份必须大于1878年。");
        }

        // 3. 检查是否为工作日（周一至周五）
        DayOfWeek dayOfWeek = tradedDatetime.getDayOfWeek();
        if (dayOfWeek == DayOfWeek.SATURDAY || dayOfWeek == DayOfWeek.SUNDAY) {
            throw new IllegalArgumentException("交易时间必须是工作日（周一至周五）。");
        }

        // 4. 检查交易时间范围（9:00 - 15:30）
        LocalTime tradedTime = tradedDatetime.toLocalTime();
        if (tradedTime.isBefore(LocalTime.of(9, 0)) || tradedTime.isAfter(LocalTime.of(15, 30))) {
            throw new IllegalArgumentException("交易时间必须在09:00至15:30之间。");
        }
    }
}


