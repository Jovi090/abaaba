import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeParseException;
import java.time.format.ResolverStyle;
import java.util.Scanner;

public class TradeRecorder {
    private static final DateTimeFormatter DATETIME_FORMATTER = DateTimeFormatter.ofPattern("uuuu-MM-dd HH:mm")
            .withResolverStyle(ResolverStyle.STRICT);
    private static final Scanner scanner = new Scanner(System.in);

    public void recordNewTrade() {
        LocalDateTime tradedDatetime;
        while (true) { 
            try {
                System.out.print("取引日時を入力してください (yyyy-MM-dd HH:mm): ");
                tradedDatetime = LocalDateTime.parse(scanner.nextLine().trim(), DATETIME_FORMATTER);
                System.out.println("输入的日期时间: " + tradedDatetime);
                break; // 若输入的日期时间有效，则跳出循环
            } catch (DateTimeParseException e) {
                System.out.println("无效的日期时间。请按正确格式重新输入。");
            }
        }
    }
}
