function onEdit(e) {
  const sheet = e.source.getActiveSheet();
  const range = e.range;

  if (range.getColumn() === 5 && range.getRow() >= 5 && range.getValue() === true) {
    const row = range.getRow();

    const deadline = sheet.getRange(row, 3).getValue();
    let weekdayValue = sheet.getRange(row, 4).getValue();
    const calendarId = "c_f0b5d959f3cc32e2044da773e557ad89fbe5944b09e296a7a8b819b8b6387753@group.calendar.google.com";

    const weekdays = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
    let weekdayName = "";

    if (!isNaN(weekdayValue)) {
      const index = parseInt(weekdayValue, 10) - 1;
      weekdayName = weekdays[index] || "Invalid Day";
      sheet.getRange(row, 4).setValue(weekdayName);
    } else {
      weekdayName = weekdayValue;
    }

    const calendar = CalendarApp.getCalendarById(calendarId);
    const eventTitle = `Weekly Partnerships Updates Reminder: ${weekdayName}`;
    const eventDescription = `Kindly be reminded to accomplish your weekly partnerships updates on or before ${weekdayName}!`;
    const event = calendar.createAllDayEvent(eventTitle, new Date(deadline), { description: eventDescription });

    const botToken = " ";
    const chatId = "-1003136077841"; 
    const messageThreadId = 2;       
    const docUrl = "http://bit.ly/3L6uD71";

    const formattedDate = new Date(deadline).toLocaleDateString("en-PH", {
      year: "numeric",
      month: "long",
      day: "numeric",
      timeZone: "Asia/Manila"
    });

    const message = `*Good morning, LAs!*\n\nKindly be reminded to accomplish your respective *weekly partnerships updates* on or before *${formattedDate} (${weekdayName})*:\n\n🔗 ${docUrl}\n\nThank you so much and have a lovely weekend! 😽💌\n\ncc: @vernicerecrio`;

    const url = `https://api.telegram.org/bot${botToken}/sendMessage`;
    const payload = {
      chat_id: chatId,
      message_thread_id: messageThreadId,
      text: message,
      parse_mode: "Markdown"
    };

    try {
      const response = UrlFetchApp.fetch(url, {
        method: "post",
        contentType: "application/json",
        payload: JSON.stringify(payload),
        muteHttpExceptions: true
      });

      const result = JSON.parse(response.getContentText());

      if (result.ok) {
        const successCell = sheet.getRange(row, 6);
        successCell.setValue("Yes");
        successCell.setFontColor("black");

        sheet.getRange(row, 7).clearContent();

      } else {
        const failCell = sheet.getRange(row, 6);
        failCell.setValue("Error");
        failCell.setFontColor("red");

        sheet.getRange(row, 7).setValue(result.description || "Unknown error");
      }

    } catch (err) {
      const failCell = sheet.getRange(row, 6);
      failCell.setValue("Error");
      failCell.setFontColor("red");

      sheet.getRange(row, 7).setValue(err.message || err.toString());
    }
  }
}
