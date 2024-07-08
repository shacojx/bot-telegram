import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

public class Main {
    public static void main(String[] args) {
        // Định dạng ngày giờ
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        // Chuỗi ngày giờ
        String date1 = "2024-07-07 23:51:36";
        String date2 = "2024-07-07 23:55:36";

        try {
            // Phân tích chuỗi ngày giờ thành đối tượng Date
            Date startDate = dateFormat.parse(date1);
            Date endDate = dateFormat.parse(date2);

            // Lấy thời gian tính bằng mili giây
            long startTime = startDate.getTime();
            long endTime = endDate.getTime();

            // Tính toán sự chênh lệch thời gian tính bằng giây
            long differenceInSeconds = (endTime - startTime) / 1000;

            // In ra kết quả
            System.out.println("Số giây giữa " + date1 + " và " + date2 + " là: " + differenceInSeconds);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
}
