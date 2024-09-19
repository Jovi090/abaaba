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
