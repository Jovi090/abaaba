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

    // 验证交易时间格式和逻辑
    public void setTradedDatetime(LocalDateTime tradedDatetime) {
        if (isValidDatetime(tradedDatetime)) {
            this.tradedDatetime = tradedDatetime;
        } else {
            throw new IllegalArgumentException("无效的交易时间，请输入1878年之后且不晚于当前时间的有效日期。");
        }
    }

    public static boolean isValidDatetime(LocalDateTime tradedDatetime) {
        LocalDateTime minDate = LocalDateTime.of(1878, 1, 1, 0, 0);
        LocalDateTime now = LocalDateTime.now();
        return tradedDatetime != null && tradedDatetime.isAfter(minDate) && tradedDatetime.isBefore(now);
    }

    // Getter 和 Setter 方法
    public LocalDateTime getTradedDatetime() {
        return tradedDatetime;
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

    // 验证买卖方向
    private static boolean isValidSide(String side) {
        return "Buy".equalsIgnoreCase(side) || "Sell".equalsIgnoreCase(side);
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

    // 验证数量
    private static boolean isValidQuantity(int quantity) {
        return quantity > 0 && quantity % 100 == 0;
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

    // 验证交易单价
    private static boolean isValidUnitPrice(BigDecimal tradedUnitPrice) {
        return tradedUnitPrice != null && tradedUnitPrice.scale() <= 2 && tradedUnitPrice.compareTo(BigDecimal.ZERO) > 0;
    }

    public LocalDateTime getInputDatetime() {
        return inputDatetime;
    }

    public void setInputDatetime(LocalDateTime inputDatetime) {
        this.inputDatetime = inputDatetime;
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
