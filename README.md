package simplex.bn25.yourWindowsAccount.trading;

public class Stock {
    private String ticker;
    private String productName;
    private String market;
    private long sharesIssued;

    public Stock(String ticker, String productName, String market, long sharesIssued) {
        this.ticker = ticker;
        this.productName = productName;
        this.market = market;
        this.sharesIssued = sharesIssued;
    }

    public String getTicker() {
        return ticker;
    }

    public String getProductName() {
        return productName;
    }

    public String getMarket() {
        return market;
    }

    public long getSharesIssued() {
        return sharesIssued;
    }
}



package simplex.bn25.yourWindowsAccount.trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.text.NumberFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class StockMasterDisplay {

    private static final String CSV_FILE_PATH = "path/to/your/csvfile.csv"; // CSVファイルのパスを設定してください
    private static final int TICKER_WIDTH = 6;
    private static final int PRODUCT_NAME_WIDTH = 30;
    private static final int MARKET_WIDTH = 8;
    private static final int SHARES_ISSUED_WIDTH = 15;

    public void displayStockMasterList() {
        List<Stock> stockData = readStockData();

        if (stockData == null) {
            System.out.println("銘柄マスタデータの読み込みに失敗しました。");
            return;
        }

        // ヘッダーを表示
        printHeader();

        // 各行のデータを表示
        for (Stock stock : stockData) {
            printStockRow(stock);
        }

        // 区切り線を表示
        printSeparator();
    }

    private List<Stock> readStockData() {
        List<Stock> stockData = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(CSV_FILE_PATH))) {
            String line;
            br.readLine(); // ヘッダー行をスキップ
            while ((line = br.readLine()) != null) {
                String[] values = line.split(",");
                if (values.length == 4) {
                    String ticker = values[0].trim();
                    String productName = values[1].trim();
                    String marketCode = values[2].trim();
                    long sharesIssued = Long.parseLong(values[3].trim());

                    Stock stock = new Stock(ticker, productName, convertMarketCode(marketCode), sharesIssued);
                    stockData.add(stock);
                }
            }
        } catch (IOException | NumberFormatException e) {
            System.err.println("CSVファイルの読み込み中にエラーが発生しました: " + e.getMessage());
            return null;
        }

        return stockData;
    }

    private void printHeader() {
        printSeparator();
        System.out.printf("| %-"+TICKER_WIDTH+"s | %-"+PRODUCT_NAME_WIDTH+"s | %-"+MARKET_WIDTH+"s | %"+SHARES_ISSUED_WIDTH+"s |%n",
                "Ticker", "Product Name", "Market", "Shares Issued");
        printSeparator();
    }

    private void printStockRow(Stock stock) {
        String ticker = stock.getTicker();
        String productName = abbreviate(stock.getProductName());
        String market = stock.getMarket();
        String sharesIssued = formatNumber(stock.getSharesIssued());

        System.out.printf("| %-"+TICKER_WIDTH+"s | %-"+PRODUCT_NAME_WIDTH+"s | %-"+MARKET_WIDTH+"s | %"+SHARES_ISSUED_WIDTH+"s |%n",
                ticker, productName, market, sharesIssued);
    }

    private String abbreviate(String text) {
        return text.length() > PRODUCT_NAME_WIDTH ? text.substring(0, PRODUCT_NAME_WIDTH - 3) + "..." : text;
    }

    private String convertMarketCode(String code) {
        switch (code) {
            case "P":
                return "Prime";
            case "S":
                return "Standard";
            case "G":
                return "Growth";
            default:
                return "Unknown";
        }
    }

    private String formatNumber(long number) {
        return NumberFormat.getNumberInstance(Locale.US).format(number);
    }

    private void printSeparator() {
        System.out.println("==========================================================================");
    }

    public static void main(String[] args) {
        StockMasterDisplay display = new StockMasterDisplay();
        display.displayStockMasterList();
    }
}




