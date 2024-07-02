# Bot telegram - share phone

Để cho phép người dùng chia sẻ số điện thoại với bot và sau đó bot gọi đến API để xử lý thông tin này, bạn có thể làm như sau:

### Bước 1: Cấu hình bot để nhận số điện thoại từ người dùng

1. **Thêm hàm xử lý số điện thoại vào mã bot:**

   Đầu tiên, bạn cần thêm một handler để xử lý khi người dùng chia sẻ số điện thoại với bot. Telegram hỗ trợ lệnh `/start` và nút chia sẻ số điện thoại để dễ dàng cho người dùng chia sẻ thông tin này.

   ```python
   from telegram import ReplyKeyboardMarkup, ReplyKeyboardRemove
   from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, ConversationHandler
   import requests
   from dotenv import load_dotenv
   import os

   # Load environment variables
   load_dotenv()

   # Get your Telegram bot token from environment variables
   TOKEN = os.getenv('TELEGRAM_BOT_TOKEN')

   # Define states for the conversation
   PHONE, CONFIRM = range(2)

   # Define a command handler for the /start command
   def start(update, context):
       update.message.reply_text(
           'Xin chào! Gửi cho tôi số điện thoại của bạn để chia sẻ với API.'
           ' Nếu không muốn chia sẻ, có thể bấm /cancel để hủy.'
       )
       return PHONE

   # Define a function to handle the phone number input
   def receive_phone(update, context):
       phone_number = update.message.contact.phone_number
       context.user_data['phone_number'] = phone_number
       update.message.reply_text(f'Bạn đã chia sẻ số điện thoại: {phone_number}.')
       update.message.reply_text('Bạn có muốn tiếp tục không? (yes/no)')
       return CONFIRM

   # Define a function to confirm if user wants to continue
   def confirm(update, context):
       text = update.message.text.lower()
       if text == 'yes':
           # Call your API with the phone number
           phone_number = context.user_data['phone_number']
           api_url = 'https://your-api-endpoint.com/process_phone_number'
           response = requests.post(api_url, json={'phone_number': phone_number})

           if response.status_code == 200:
               api_response = response.json()
               result = api_response.get('result', 'Không có kết quả.')
               update.message.reply_text(f'Kết quả từ API: {result}')
           else:
               update.message.reply_text('Đã xảy ra lỗi khi gọi API.')

       elif text == 'no':
           update.message.reply_text('Bạn đã hủy bỏ.')
       else:
           update.message.reply_text('Xin vui lòng trả lời "yes" hoặc "no".')
       return ConversationHandler.END

   def cancel(update, context):
       update.message.reply_text('Bạn đã hủy bỏ.')
       return ConversationHandler.END

   def main():
       updater = Updater(TOKEN, use_context=True)

       # Get the dispatcher to register handlers
       dp = updater.dispatcher

       # Create a conversation handler with states
       conv_handler = ConversationHandler(
           entry_points=[CommandHandler('start', start)],
           states={
               PHONE: [MessageHandler(Filters.contact, receive_phone)],
               CONFIRM: [MessageHandler(Filters.text & ~Filters.command, confirm)],
           },
           fallbacks=[CommandHandler('cancel', cancel)],
       )

       # Add conversation handler to dispatcher
       dp.add_handler(conv_handler)

       # Start the Bot
       updater.start_polling()

       # Run the bot until you press Ctrl-C
       updater.idle()

   if __name__ == '__main__':
       main()
   ```

### Bước 2: Giải thích mã bot

1. **Hàm `start`:**
   - Được gọi khi người dùng bắt đầu cuộc trò chuyện với bot. Nó yêu cầu người dùng chia sẻ số điện thoại và chuyển trạng thái sang `PHONE`.

2. **Hàm `receive_phone`:**
   - Xử lý khi người dùng chia sẻ số điện thoại. Nó sử dụng `Filters.contact` để chấp nhận số điện thoại chia sẻ từ người dùng và lưu vào `context.user_data`.

3. **Hàm `confirm`:**
   - Xác nhận lại từ người dùng xem họ có muốn tiếp tục xử lý với số điện thoại đã chia sẻ hay không. Nếu người dùng đồng ý (`yes`), bot sẽ gọi API với số điện thoại và xử lý kết quả trả về.

4. **Hàm `cancel`:**
   - Cho phép người dùng hủy bỏ quá trình.

### Bước 3: Triển khai và chạy bot

1. **Triển khai mã bot lên một máy chủ (server) hoặc dịch vụ cloud.**
2. **Chạy mã bot:**
   ```bash
   python bot.py
   ```

### Bước 4: Kiểm tra bot trên Telegram

1. **Mở Telegram và tìm kiếm tên người dùng của bot bạn đã tạo.**
2. **Gửi lệnh `/start` để bắt đầu và chia sẻ số điện thoại khi được yêu cầu.**

Bot sẽ gọi đến API của bạn với số điện thoại mà người dùng đã chia sẻ và xử lý kết quả như đã định nghĩa trong hàm `confirm`. Đảm bảo rằng bạn đã cung cấp đúng token của bot và endpoint của API trong mã của bạn.
