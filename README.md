// 新建 Side.java
public enum Side {
    BUY("Buy"),
    SELL("Sell");

    private final String value;

    Side(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    // 静态方法用于验证和获取枚举实例
    public static Side fromString(String side) {
        for (Side s : Side.values()) {
            if (s.value.equalsIgnoreCase(side)) {
                return s;
            }
        }
        throw new IllegalArgumentException("無効な売買区分です。BuyまたはSellを入力してください。");
    }
}

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class Trade {
    private LocalDateTime tradedDatetime;
    private String ticker;
    private String tickerName;
    private Side side;  // 将 side 类型改为 Side 枚举
    private int quantity;
    private BigDecimal tradedUnitPrice;
    private LocalDateTime inputDatetime;

    public static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

    public Trade(LocalDateTime tradedDatetime, String ticker, String tickerName, String side, int quantity, BigDecimal tradedUnitPrice, LocalDateTime inputDatetime) {
        this.tradedDatetime = tradedDatetime;
        this.ticker = ticker;
        this.tickerName = tickerName;
        this.side = Side.fromString(side);  // 使用 Side.fromString 进行验证和赋值
        this.quantity = quantity;
        this.tradedUnitPrice = tradedUnitPrice;
        this.inputDatetime = inputDatetime;
    }

    public Side getSide() {
        return side;
    }

    public void setSide(String side) {
        this.side = Side.fromString(side);  // 使用 Side.fromString 进行验证和赋值
    }

    // 其他现有方法保持不变
}

public void recordNewTrade() {
    LocalDateTime tradedDatetime = null;
    while (true) {
        try {
            System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm): ");
            tradedDatetime = LocalDateTime.parse(scanner.nextLine().trim(), DATETIME_FORMATTER);

            if (!Trade.isValidTradeDateTime(tradedDatetime)) {
                System.out.println("無効な取引日時です。1878年以降の平日の9:00〜15:30までの有効な日時を入力してください。");
                continue;
            }
            break;
        } catch (DateTimeParseException e) {
            System.out.println("取引日時の形式が正しくありません。再度入力してください。");
        }
    }

    String ticker;
    while (true) {
        System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
        ticker = scanner.nextLine().trim();

        if (!Stock.isValidTicker(ticker)) {
            System.out.println("無効な銘柄コードです。再度入力してください。");
            continue;
        }

        if (!tradeRepository.isTickerRegistered(ticker)) {
            System.out.println("銘柄コードが存在しません。再度入力してください。");
        } else {
            break;
        }
    }

    String tickerName;
    while (true) {
        System.out.print("銘柄名を入力してください: ");
        tickerName = scanner.nextLine().trim();
        if (Stock.isValidProductName(tickerName)) {
            break;
        } else {
            System.out.println("無効な銘柄名です。再度入力してください。");
        }
    }

    Side side;
    while (true) {
        System.out.print("売買区分を入力してください (Buy/Sell): ");
        String inputSide = scanner.nextLine().trim();
        try {
            side = Side.fromString(inputSide);  // 使用 Side.fromString 进行验证和赋值
            break;
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }

    int quantity;
    while (true) {
        System.out.print("数量を入力してください (100株単位): ");
        try {
            quantity = Integer.parseInt(scanner.nextLine().trim());
            if (Trade.isValidQuantity(quantity)) {
                break;
            } else {
                System.out.println("数量は100の倍数の正の整数を入力してください。");
            }
        } catch (NumberFormatException e) {
            System.out.println("数量の形式が正しくありません。再度入力してください。");
        }
    }

    BigDecimal tradedUnitPrice;
    while (true) {
        System.out.print("取引単価を入力してください (小数点以下2桁まで): ");
        try {
            tradedUnitPrice = new BigDecimal(scanner.nextLine().trim());
            if (Trade.isValidTradedUnitPrice(tradedUnitPrice)) {
                break;
            } else {
                System.out.println("取引単価は小数点以下2桁以内で入力してください。");
            }
        } catch (NumberFormatException e) {
            System.out.println("取引単価の形式が正しくありません。再度入力してください。");
        }
    }

    LocalDateTime inputDatetime = LocalDateTime.now();
    Trade trade = new Trade(tradedDatetime, ticker, tickerName, side.getValue(), quantity, tradedUnitPrice, inputDatetime);

    if (tradeRepository.saveTrade(trade)) {
        tradeView.showTradeAddedMessage(trade);
    } else {
        System.out.print("データの書き込みにエラーが発生しました。");
    }
}
