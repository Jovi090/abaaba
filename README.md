import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.Scanner;
import java.util.regex.Pattern;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        System.out.print("请输入变量a：");
        String a = scanner.nextLine();
        
        // 定义正则表达式匹配四位数，前三位为数字，第四位为英文或数字
        String regex = "^[0-9]{3}[a-zA-Z0-9]$";
        
        // 判断输入格式是否正确
        if (!Pattern.matches(regex, a)) {
            System.out.println("格式有误");
            return;
        }
        
        // 检查CSV文件中的第一列是否存在相同的值
        String csvFile = "yourfile.csv"; // 将此处替换为你的CSV文件路径
        String line;
        String csvSplitBy = ","; // 假设CSV文件使用逗号分隔

        try (BufferedReader br = new BufferedReader(new FileReader(csvFile))) {
            while ((line = br.readLine()) != null) {
                // 获取CSV文件的第一列
                String[] data = line.split(csvSplitBy);
                if (data[0].equals(a)) {
                    System.out.println("重复的代码");
                    return;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        
        System.out.println("输入的代码是唯一的。");
    }
}
