package trading;

public class CsvFilePaths {

    // 定义两个静态常量，用于存储CSV文件路径
    public static final String STOCK_CSV_FILE_PATH = "C:/Users/qqq/Desktop/Stock.csv";
    public static final String TRADE_CSV_FILE_PATH = "C:/Users/qqq/Desktop/Trade.csv";

    // 私有构造函数，防止实例化该类
    private CsvFilePaths() {
        // 防止该类被实例化
    }
}

package trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.text.NumberFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class StockMasterDisplay {

    // 使用 CsvFilePaths 中定义的路径
    private static final String CSV_FILE_PATH = CsvFilePaths.STOCK_CSV_FILE_PATH;
    // 其他代码保持不变...
}

package trading;

import java.io.FileWriter;
import java.io.IOException;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Scanner;

public class TradeEntry {

    // 使用 CsvFilePaths 中定义的路径
    private static final String TRADE_CSV_FILE_PATH = CsvFilePaths.TRADE_CSV_FILE_PATH;
    // 其他代码保持不变...
}

package trading;

public enum MarketType {
    STANDARD("Standard Market"),
    PRIME("Prime Market"),
    GROWTH("Growth Market");

    private final String description;

    MarketType(String description) {
        this.description = description;
    }

    public String getDescription() {
        return description;
    }
}

package trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.text.NumberFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class StockMasterDisplay {

    private List<StockMasterEntry> stockMasterEntries;
    private MarketType marketType;

    // 修改构造函数，接受MarketType
    public StockMasterDisplay(MarketType marketType) {
        this.stockMasterEntries = new ArrayList<>();
        this.marketType = marketType;
    }

    // 如果不想修改原有构造函数，提供一个setMarketType方法
    public void setMarketType(MarketType marketType) {
        this.marketType = marketType;
    }

    public MarketType getMarketType() {
        return marketType;
    }

    // 显示市场信息
    public void displayMarketInfo() {
        System.out.println("Selected Market: " + marketType.getDescription());
    }

    // 其他原本的功能保持不变，如读取CSV、解析数据等
}

package trading;

public class StockMasterEntry {

    private String stockName;
    private double stockPrice;
    private int quantity;

    // 构造函数
    public StockMasterEntry(String stockName, double stockPrice, int quantity) {
        this.stockName = stockName;
        this.stockPrice = stockPrice;
        this.quantity = quantity;
    }

    // Getter和Setter方法
    public String getStockName() {
        return stockName;
    }

    public void setStockName(String stockName) {
        this.stockName = stockName;
    }

    public double getStockPrice() {
        return stockPrice;
    }

    public void setStockPrice(double stockPrice) {
        this.stockPrice = stockPrice;
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }

    // 其他方法保持不变
}
