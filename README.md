11111111111111111111
package trading;

import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        StockMasterDisplay stockMasterDisplay = new StockMasterDisplay();
        TradeListDisplay tradeListDisplay = new TradeListDisplay();
        boolean running = true;

        // アプリケーションの開始メッセージ
        System.out.println("株式取引管理システムを開始します。");

        while (running) {
            // メニューを表示
            System.out.println("操作するメニューを選んでください。");
            System.out.println("　1. 銘柄マスタ一覧表示");
            System.out.println("　2. 銘柄マスタ新規登録");
            System.out.println("　3. 取引新規登録");
            System.out.println("　4. 取引一覧表示");
            System.out.println("　9. アプリケーションを終了する");
            System.out.print("入力してください: ");

            // ユーザー入力を受け取る
            String input = scanner.nextLine();

            // 入力に応じた処理を実行
            switch (input) {
                case "1":
                    System.out.println("「銘柄マスタ一覧表示」が選択されました。");
                    stockMasterDisplay.displayStockMasterList();
                    break;
                case "2":
                    System.out.println("「銘柄マスタ新規登録」が選択されました。");
                    StockMasterEntry.registerNewStock();
                    break;
                case "3":
                    System.out.println("「取引新規登録」が選択されました。");
                    TradeEntry.recordTrade();
                    break;
                case "4":
                    System.out.println("「取引一覧表示」が選択されました。");
                    tradeListDisplay.displayTradeList();
                    break;
                case "9":
                    System.out.println("アプリケーションを終了します。");
                    running = false;
                    break;
                default:
                    System.out.println("\"" + input + "\" に対応するメニューは存在しません。");
                    break;
            }

            // メニューに戻るための区切り
            System.out.println("--");
        }

        // リソースの解放
        scanner.close();
    }
}

222222222222222222222222222222222

package trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.text.NumberFormat;
import java.util.ArrayList;
import java.util.List;
import java.util.Locale;

public class StockMasterDisplay {

    private static final String CSV_FILE_PATH = "C:/Users/qqq/Desktop/Stock.csv"; // CSVファイルのパスを設定してください
    private static final int TICKER_WIDTH = 6;
    private static final int PRODUCT_NAME_WIDTH = 30;
    private static final int MARKET_WIDTH = 8;
    private static final int SHARES_ISSUED_WIDTH = 15;

    public void displayStockMasterList() {
        List<String[]> stockData = readStockData();

        if (stockData == null) {
            System.out.println("銘柄マスタデータの読み込みに失敗しました。");
            return;
        }

        // ヘッダーを表示
        printHeader();

        // 各行のデータを表示
        for (String[] stock : stockData) {
            printStockRow(stock);
        }

        // 区切り線を表示
        printSeparator();
    }

    private List<String[]> readStockData() {
        List<String[]> stockData = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(CSV_FILE_PATH))) {
            String line;
            br.readLine(); // ヘッダー行をスキップ
            while ((line = br.readLine()) != null) {
                String[] values = line.split(",");
                if (values.length == 4) {
                    stockData.add(values);
                }
            }
        } catch (IOException e) {
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

    private void printStockRow(String[] stock) {
        String ticker = stock[0].trim();
        String productName = abbreviate(stock[1].trim());
        String market = convertMarketCode(stock[2].trim());
        String sharesIssued = formatNumber(stock[3].trim());

        System.out.printf("| %-"+TICKER_WIDTH+"s | %-"+PRODUCT_NAME_WIDTH+"s | %-"+MARKET_WIDTH+"s | %"+SHARES_ISSUED_WIDTH+"s |%n",
                ticker, productName, market, sharesIssued);
    }

    private String abbreviate(String text) {
        return text.length() > PRODUCT_NAME_WIDTH ? text.substring(0, PRODUCT_NAME_WIDTH - 3) + "..." : text;
    }

    private String convertMarketCode(String code) {
        return switch (code) {
            case "P" -> "Prime";
            case "S" -> "Standard";
            case "G" -> "Growth";
            default -> "Unknown";
        };
    }

    private String formatNumber(String numberStr) {
        try {
            long number = Long.parseLong(numberStr);
            return NumberFormat.getNumberInstance(Locale.US).format(number);
        } catch (NumberFormatException e) {
            return "N/A";
        }
    }

    private void printSeparator() {
        System.out.println("========================================================================");
    }

    public static void main(String[] args) {
        StockMasterDisplay display = new StockMasterDisplay();
        display.displayStockMasterList();
    }
}


333333333333333333333333333333

package trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.HashSet;
import java.util.Scanner;
import java.util.Set;

public class StockMasterEntry {

    private static final String CSV_FILE_PATH = "C:/Users/qqq/Desktop/Stock.csv";
    private static final Set<Character> VALID_LETTERS = new HashSet<>();
    static {
        for (char c = 'A'; c <= 'Z'; c++) {
            if (c != 'B' && c != 'E' && c != 'I' && c != 'O' && c != 'Q' && c != 'V' && c != 'Z') {
                VALID_LETTERS.add(c);
            }
        }
    }

    public static void registerNewStock() {
        Scanner scanner = new Scanner(System.in);

        String ticker;
        while (true) {
            System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
            ticker = scanner.nextLine().trim().toUpperCase();
            if (isValidTicker(ticker) && !isTickerRegistered(ticker)) {
                break;
            } else {
                System.out.println("無効な銘柄コードです、または既に登録されています。もう一度入力してください。");
            }
        }

        String productName;
        while (true) {
            System.out.print("銘柄名を入力してください: ");
            productName = scanner.nextLine().trim();
            if (isValidProductName(productName)) {
                break;
            } else {
                System.out.println("無効な銘柄名です。もう一度入力してください。");
            }
        }

        String market;
        while (true) {
            System.out.print("上場市場を入力してください: ");
            market = scanner.nextLine().trim();
            if (isValidMarket(market)) {
                break;
            } else {
                System.out.println("無効な上場市場です。もう一度入力してください。");
            }
        }

        long sharesIssued;
        while (true) {
            System.out.print("発行済み株式数を入力してください: ");
            try {
                sharesIssued = Long.parseLong(scanner.nextLine().trim());
                if (isValidSharesIssued(sharesIssued)) {
                    break;
                } else {
                    System.out.println("無効な株式数です。もう一度入力してください。");
                }
            } catch (NumberFormatException e) {
                System.out.println("数値として認識できません。もう一度入力してください。");
            }
        }

        // 上場市場をP, S, Gのコードに変換
        String marketCode = convertMarketToCode(market);

        // CSVファイルに追記
        try (FileWriter writer = new FileWriter(CSV_FILE_PATH, true)) {
            writer.write(String.format("\n%s,%s,%s,%d", ticker, productName, marketCode, sharesIssued));
            System.out.println(productName + " を新規登録しました。");
        } catch (IOException e) {
            System.out.println("CSVファイルへの書き込み中にエラーが発生しました: " + e.getMessage());
        }
    }

    private static boolean isValidTicker(String ticker) {
        return ticker.matches("^[0-9][A-Z0-9][0-9][A-Z0-9]$")
                && isValidTickerDigits(ticker);
    }

    private static boolean isValidTickerDigits(String ticker) {
        // 第1桁と第3桁が数字であり、第2桁と第4桁が有効な英字または数字であることを確認する
        return Character.isDigit(ticker.charAt(0))
                && Character.isDigit(ticker.charAt(2))
                && (Character.isDigit(ticker.charAt(1)) || VALID_LETTERS.contains(ticker.charAt(1)))
                && (Character.isDigit(ticker.charAt(3)) || VALID_LETTERS.contains(ticker.charAt(3)));
    }

    private static boolean isTickerRegistered(String ticker) {
        try (BufferedReader reader = new BufferedReader(new FileReader(CSV_FILE_PATH))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields[0].equals(ticker)) {
                    return true;
                }
            }
        } catch (IOException e) {
            System.out.println("CSVファイルの読み込み中にエラーが発生しました: " + e.getMessage());
        }
        return false;
    }

    private static boolean isValidProductName(String productName) {
        return productName.matches("^[A-Za-z0-9 .()]+$");
    }

    private static boolean isValidMarket(String market) {
        String marketUpper = market.trim().toUpperCase();
        return marketUpper.equals("PRIME") || marketUpper.equals("STANDARD") || marketUpper.equals("GROWTH")
                || marketUpper.equals("プライム") || marketUpper.equals("スタンダード") || marketUpper.equals("グロース");
    }

    private static boolean isValidSharesIssued(long sharesIssued) {
        return sharesIssued >= 1 && sharesIssued <= 999_999_999_999L;
    }

    private static String convertMarketToCode(String market) {
        return switch (market.trim().toUpperCase()) {
            case "PRIME", "プライム" -> "P";
            case "STANDARD", "スタンダード" -> "S";
            case "GROWTH", "グロース" -> "G";
            default -> "";
        };
    }
}


444444444444444444

package trading;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public record Trade(LocalDateTime tradedDatetime, String ticker, String tickername, String side, int quantity, BigDecimal tradedUnitPrice,
                    LocalDateTime inputDatetime) {
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    // toCSVFormat メソッドの追加
    public String toCSVFormat() {
        return String.join(",",
                tradedDatetime.format(DATETIME_FORMATTER),
                ticker,
                tickername,
                side,
                String.valueOf(quantity),
                tradedUnitPrice.toString(),
                inputDatetime.format(DATETIME_FORMATTER));
    }
}

package trading;

import java.io.FileWriter;
import java.io.IOException;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Scanner;

public class TradeEntry {

    private static final String TRADE_CSV_FILE_PATH = "C:/Users/qqq/Desktop/Trade.csv";
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public static void recordTrade() {
        Scanner scanner = new Scanner(System.in);

        // 1. 取引日時の入力
        LocalDateTime tradedDatetime;
        while (true) {
            System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm): ");
            String tradedDatetimeStr = scanner.nextLine().trim();
            try {
                tradedDatetime = LocalDateTime.parse(tradedDatetimeStr, DATETIME_FORMATTER);
                if (isValidTradeDatetime(tradedDatetime)) {
                    break;
                } else {
                    System.out.println("無効な取引日時です。再度入力してください。");
                }
            } catch (Exception e) {
                System.out.println("日時の形式が無効です。再度入力してください。");
            }
        }

        // 2. 銘柄コードの入力
        String ticker;
        while (true) {
            System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
            ticker = scanner.nextLine().trim().toUpperCase();
            if (!isValidTicker(ticker)) {
                System.out.println("無効な銘柄コードです。再度入力してください。");
            }  else {
                break;
            }
        }

        String tickername;
        while (true) {
            System.out.print("銘柄名を入力してください: ");
            tickername = scanner.nextLine().trim();
            if (isValidTickerName(tickername)) {
                break;
            } else {
                System.out.println("無効な銘柄名です。もう一度入力してください。");
            }
        }

        // 3. 売買区分の入力
        String side;
        while (true) {
            System.out.print("売買区分を入力してください (Buy/Sell): ");
            side = scanner.nextLine().trim();
            if (isValidSide(side)) {
                break;
            } else {
                System.out.println("無効な売買区分です。再度入力してください。");
            }
        }

        // 4. 数量の入力
        int quantity;
        while (true) {
            System.out.print("数量を入力してください (100株単位): ");
            try {
                quantity = Integer.parseInt(scanner.nextLine().trim());
                if (isValidQuantity(quantity)) {
                    break;
                } else {
                    System.out.println("無効な数量です。再度入力してください。");
                }
            } catch (NumberFormatException e) {
                System.out.println("数値として認識できません。再度入力してください。");
            }
        }

        // 5. 取引単価の入力
        BigDecimal tradedUnitPrice;
        while (true) {
            System.out.print("取引単価を入力してください (小数点以下2桁まで): ");
            try {
                tradedUnitPrice = new BigDecimal(scanner.nextLine().trim());
                if (isValidUnitPrice(tradedUnitPrice)) {
                    break;
                } else {
                    System.out.println("無効な取引単価です。再度入力してください。");
                }
            } catch (NumberFormatException e) {
                System.out.println("数値として認識できません。再度入力してください。");
            }
        }

        LocalDateTime inputDatetime = LocalDateTime.now();
        Trade trade = new Trade(tradedDatetime, ticker, tickername, side, quantity, tradedUnitPrice, inputDatetime);

        // 6. 取引データのCSVへの書き込み
        try (FileWriter writer = new FileWriter(TRADE_CSV_FILE_PATH, true)) {
            writer.write(trade.toCSVFormat() + "\n");
            System.out.println("取引データを新規登録しました。");
        } catch (IOException e) {
            System.out.println("CSVファイルへの書き込み中にエラーが発生しました: " + e.getMessage());
        }
    }

    private static boolean isValidTradeDatetime(LocalDateTime tradedDatetime) {
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime startOfDay = tradedDatetime.toLocalDate().atTime(9, 0);
        LocalDateTime endOfDay = tradedDatetime.toLocalDate().atTime(15, 30);
        return tradedDatetime.isBefore(now) && (tradedDatetime.isEqual(startOfDay) || tradedDatetime.isAfter(startOfDay))
                && (tradedDatetime.isEqual(endOfDay) || tradedDatetime.isBefore(endOfDay))
                && tradedDatetime.getDayOfWeek().getValue() <= 5; // 月〜金
    }

    private static boolean isValidTicker(String ticker) {
        return ticker.matches("^[0-9][A-Z0-9][0-9][A-Z0-9]$");
    }

    private static boolean isValidTickerName(String productName) {
        return productName.matches("^[A-Za-z0-9 .()]+$");
    }

    private static boolean isValidSide(String side) {
        return side.equalsIgnoreCase("Buy") || side.equalsIgnoreCase("Sell");
    }

    private static boolean isValidQuantity(int quantity) {
        return quantity > 0 && quantity % 100 == 0;
    }

    private static boolean isValidUnitPrice(BigDecimal unitPrice) {
        return unitPrice.scale() <= 2;
    }
}


5555555555555555555555

package trading;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

public class TradeListDisplay {

    private static final String CSV_FILE_PATH = "C:/Users/qqq/Desktop/Trade.csv";
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
    private static final int DATETIME_WIDTH = 19;   // 取引日時の幅
    private static final int TICKER_WIDTH = 7;      // 銘柄コードの幅
    private static final int TICKERNAME_WIDTH = 30; // 銘柄名の幅
    private static final int SIDE_WIDTH = 4;        // 売買区分の幅
    private static final int QUANTITY_WIDTH = 10;   // 数量の幅
    private static final int UNIT_PRICE_WIDTH = 12; // 取引単価の幅

    public void displayTradeList() {
        List<Trade> tradeData = readTradeData();

        if (tradeData == null || tradeData.isEmpty()) {
            System.out.println("取引データが存在しません。");
            return;
        }

        // 取引日時の降順でソート
        tradeData.sort(Comparator.comparing(Trade::tradedDatetime).reversed());

        // ヘッダーを表示
        printHeader();

        // 各行のデータを表示
        for (Trade trade : tradeData) {
            printTradeRow(trade);
        }

        // 区切り線を表示
        printSeparator();
    }

    private List<Trade> readTradeData() {
        List<Trade> tradeData = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(new FileReader(CSV_FILE_PATH))) {
            String line;
            br.readLine(); // ヘッダー行をスキップ
            while ((line = br.readLine()) != null) {
                String[] values = line.split(",");
                if (values.length == 7) { // 取引データのカラム数は7に変更
                    LocalDateTime tradedDatetime = LocalDateTime.parse(values[0], DATETIME_FORMATTER);
                    String ticker = values[1];
                    String tickername = values[2];
                    String side = values[3];
                    int quantity = Integer.parseInt(values[4]);
                    BigDecimal tradedUnitPrice = new BigDecimal(values[5]);
                    LocalDateTime inputDatetime = LocalDateTime.parse(values[6], DATETIME_FORMATTER);
                    tradeData.add(new Trade(tradedDatetime, ticker, tickername, side, quantity, tradedUnitPrice, inputDatetime));
                }
            }
        } catch (IOException e) {
            System.err.println("CSVファイルの読み込み中にエラーが発生しました: " + e.getMessage());
            return null;
        }

        return tradeData;
    }

    private void printHeader() {
        printSeparator();
        System.out.printf("| %-"+DATETIME_WIDTH+"s | %-"+TICKER_WIDTH+"s | %-"+TICKERNAME_WIDTH+"s | %-"+SIDE_WIDTH+"s | %-"+QUANTITY_WIDTH+"s | %-"+UNIT_PRICE_WIDTH+"s |%n",
                "Trade Time", "Ticker", "Product Name", "Side", "Quantity", "Unit Price");
        printSeparator();
    }

    private void printTradeRow(Trade trade) {
        String tickername = trade.tickername();
        if (tickername.length() > TICKERNAME_WIDTH) {
            tickername = tickername.substring(0, TICKERNAME_WIDTH - 3) + "...";
        }

        System.out.printf("| %-"+DATETIME_WIDTH+"s | %-"+TICKER_WIDTH+"s | %-"+TICKERNAME_WIDTH+"s | %-"+SIDE_WIDTH+"s | %"+QUANTITY_WIDTH+"d | %"+UNIT_PRICE_WIDTH+".2f |%n",
                trade.tradedDatetime().format(DATETIME_FORMATTER),
                trade.ticker(),
                tickername,
                trade.side(),
                trade.quantity(),
                trade.tradedUnitPrice());
    }

    private void printSeparator() {
        System.out.println("=====================================================================================================");
    }
}



