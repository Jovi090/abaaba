package trading;

public enum Market {
    PRIME("P", "Prime"),
    STANDARD("S", "Standard"),
    GROWTH("G", "Growth"),
    UNKNOWN("", "Unknown");

    private final String code;
    private final String displayName;

    Market(String code, String displayName) {
        this.code = code;
        this.displayName = displayName;
    }

    public String getCode() {
        return code;
    }

    public String getDisplayName() {
        return displayName;
    }

    public static Market fromCode(String code) {
        for (Market market : Market.values()) {
            if (market.getCode().equalsIgnoreCase(code)) {
                return market;
            }
        }
        return UNKNOWN;
    }

    public static Market fromDisplayName(String displayName) {
        for (Market market : Market.values()) {
            if (market.getDisplayName().equalsIgnoreCase(displayName) ||
                market.getDisplayName().equalsIgnoreCase(convertToJapanese(displayName))) {
                return market;
            }
        }
        return UNKNOWN;
    }

    private static String convertToJapanese(String name) {
        return switch (name.toUpperCase()) {
            case "PRIME" -> "プライム";
            case "STANDARD" -> "スタンダード";
            case "GROWTH" -> "グロース";
            default -> "";
        };
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

    private static final String CSV_FILE_PATH = "C:/Users/qqq/Desktop/Stock.csv";
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
        Market market = Market.fromCode(stock[2].trim());
        String sharesIssued = formatNumber(stock[3].trim());

        System.out.printf("| %-"+TICKER_WIDTH+"s | %-"+PRODUCT_NAME_WIDTH+"s | %-"+MARKET_WIDTH+"s | %"+SHARES_ISSUED_WIDTH+"s |%n",
                ticker, productName, market.getDisplayName(), sharesIssued);
    }

    private String abbreviate(String text) {
        return text.length() > PRODUCT_NAME_WIDTH ? text.substring(0, PRODUCT_NAME_WIDTH - 3) + "..." : text;
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

        Market market;
        while (true) {
            System.out.print("上場市場を入力してください: ");
            String marketInput = scanner.nextLine().trim();
            market = Market.fromDisplayName(marketInput);
            if (market != Market.UNKNOWN) {
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

        // CSVファイルに追記
        try (FileWriter writer = new FileWriter(CSV_FILE_PATH, true)) {
            writer.write(String.format("\n%s,%s,%s,%d", ticker, productName, market.getCode(), sharesIssued));
            System.out.println(productName + " を新規登録しました。");
        } catch (IOException e) {
            System.out.println("CSVファイルへの書き込み中にエラーが発生しました: " + e.getMessage());
        }
    }

    private static boolean isValidTicker(String ticker) {
        return ticker.matches("^[0-9][A-Z0-9][0-9][A-Z0-9]$") && isValidTickerDigits(ticker);
    }

    private static boolean isValidTickerDigits(String ticker) {
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

    private static boolean isValidSharesIssued(long sharesIssued) {
        return sharesIssued >= 1 && sharesIssued <= 999_999_999_999L;
    }
}
