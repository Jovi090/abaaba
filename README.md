import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;

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
        setTradedDatetime(tradedDatetime);
        this.ticker = ticker;
        this.tickerName = tickerName;
        setSide(side);
        setQuantity(quantity);
        setTradedUnitPrice(tradedUnitPrice);
        this.inputDatetime = inputDatetime;
    }

    // Getter 和 Setter 方法
    public LocalDateTime getTradedDatetime() {
        return tradedDatetime;
    }

    public void setTradedDatetime(LocalDateTime tradedDatetime) {
        if (isValidDatetime(tradedDatetime)) {
            this.tradedDatetime = tradedDatetime;
        } else {
            throw new IllegalArgumentException("无效的交易时间，请输入1878年之后且不晚于当前时间的有效日期。");
        }
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
        if (isValidQuantity(quantity)) {
            this.quantity = quantity;
        } else {
            throw new IllegalArgumentException("数量必须为100的整数倍");
        }
    }

    public BigDecimal getTradedUnitPrice() {
        return tradedUnitPrice;
    }

    public void setTradedUnitPrice(BigDecimal tradedUnitPrice) {
        if (isValidUnitPrice(tradedUnitPrice)) {
            this.tradedUnitPrice = tradedUnitPrice;
        } else {
            throw new IllegalArgumentException("交易单价必须为小数点后最多两位的有效数值");
        }
    }

    public LocalDateTime getInputDatetime() {
        return inputDatetime;
    }

    public void setInputDatetime(LocalDateTime inputDatetime) {
        this.inputDatetime = inputDatetime;
    }

    // 验证方法

    // 验证买卖方向
    public static boolean isValidSide(String side) {
        return "Buy".equalsIgnoreCase(side) || "Sell".equalsIgnoreCase(side);
    }

    // 验证交易时间是否在1878年之后且不晚于当前时间
    public static boolean isValidDatetime(LocalDateTime tradedDatetime) {
        LocalDateTime minDate = LocalDateTime.of(1878, 1, 1, 0, 0);
        LocalDateTime now = LocalDateTime.now();
        return tradedDatetime != null && tradedDatetime.isAfter(minDate) && tradedDatetime.isBefore(now);
    }

    // 验证日期格式是否正确
    public static boolean isValidDateFormat(String dateTimeStr) {
        try {
            LocalDateTime.parse(dateTimeStr, DATETIME_FORMATTER);
            return true;
        } catch (DateTimeParseException e) {
            return false;
        }
    }

    // 验证数量是否为100的倍数
    public static boolean isValidQuantity(int quantity) {
        return quantity > 0 && quantity % 100 == 0;
    }

    // 验证交易单价是否为小数点后最多两位的有效数值
    public static boolean isValidUnitPrice(BigDecimal tradedUnitPrice) {
        return tradedUnitPrice != null && tradedUnitPrice.scale() <= 2 && tradedUnitPrice.compareTo(BigDecimal.ZERO) > 0;
    }

    // 将交易记录转换为CSV格式的字符串
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



import model.Trade;
import model.Stock;
import model.repository.TradeRepository;
import view.TradeView;

import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.Scanner;

public class TradeController {
    private final TradeRepository tradeRepository;
    private final TradeView tradeView;
    private final Scanner scanner = new Scanner(System.in);

    public TradeController(TradeRepository tradeRepository, TradeView tradeView) {
        this.tradeRepository = tradeRepository;
        this.tradeView = tradeView;
    }

    // 记录新交易
    public void recordNewTrade() {
        try {
            LocalDateTime tradedDatetime = inputTradedDatetime();
            String ticker = inputTicker();
            String tickerName = inputTickerName();
            String side = inputSide();
            int quantity = inputQuantity();
            BigDecimal tradedUnitPrice = inputTradedUnitPrice();

            LocalDateTime inputDatetime = LocalDateTime.now();
            Trade trade = new Trade(tradedDatetime, ticker, tickerName, side, quantity, tradedUnitPrice, inputDatetime);

            if (tradeRepository.saveTrade(trade)) {
                tradeView.showTradeAddedMessage(trade);
            } else {
                System.out.print("データの書き込みにエラーが発生しました。");
            }

        } catch (IllegalArgumentException e) {
            System.out.println("输入错误：" + e.getMessage());
        }
    }

    // 输入并验证交易时间
    private LocalDateTime inputTradedDatetime() {
        System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm): ");
        String dateTimeStr = scanner.nextLine().trim();

        // 检查格式
        if (!Trade.isValidDateFormat(dateTimeStr)) {
            throw new IllegalArgumentException("无效的日期格式，应为 yyyy-MM-dd HH:mm");
        }

        // 格式正确，解析并验证逻辑
        LocalDateTime tradedDatetime = LocalDateTime.parse(dateTimeStr, Trade.DATETIME_FORMATTER);
        if (!Trade.isValidDatetime(tradedDatetime)) {
            throw new IllegalArgumentException("无效的交易时间");
        }
        return tradedDatetime;
    }

    // 输入并验证股票代码
    private String inputTicker() {
        System.out.print("銘柄コードを入力してください (4桁の半角英数字): ");
        String ticker = scanner.nextLine().trim();
        if (!Stock.isValidTicker(ticker)) {
            throw new IllegalArgumentException("无效的股票代码格式");
        }
        if (!tradeRepository.isTickerRegistered(ticker)) {
            throw new IllegalArgumentException("该股票代码未找到，请检查输入或先注册该股票");
        }
        return ticker;
    }

    // 输入并验证股票名称
    private String inputTickerName() {
        System.out.print("銘柄名を入力してください: ");
        String tickerName = scanner.nextLine().trim();
        if (!Stock.isValidProductName(tickerName)) {
            throw new IllegalArgumentException("无效的股票名称格式");
        }
        if (!tradeRepository.isTickerRegistered(tickerName)) {
            throw new IllegalArgumentException("该股票名称未找到，请检查输入或先注册该股票");
        }
        return tickerName;
    }

    // 输入并验证买卖方向
    private String inputSide() {
        System.out.print("売買区分を入力してください (Buy/Sell): ");
        String side = scanner.nextLine().trim();
        if (!Trade.isValidSide(side)) {
            throw new IllegalArgumentException("无效的买卖方向，应为 'Buy' 或 'Sell'");
        }
        return side;
    }

    // 输入并验证数量
    private int inputQuantity() {
        System.out.print("数量を入力してください (100株単位): ");
        int quantity = Integer.parseInt(scanner.nextLine().trim());
        if (!Trade.isValidQuantity(quantity)) {
            throw new IllegalArgumentException("数量必须为100的整数倍");
        }
        return quantity;
    }

    // 输入并验证交易单价
    private BigDecimal inputTradedUnitPrice() {
        System.out.print("取引単価を入力してください (小数点以下2桁まで): ");
        BigDecimal tradedUnitPrice = new BigDecimal(scanner.nextLine().trim());
        if (!Trade.isValidUnitPrice(tradedUnitPrice)) {
            throw new IllegalArgumentException("交易单价必须为小数点后最多两位的有效数值");
        }
        return tradedUnitPrice;
    }
}
