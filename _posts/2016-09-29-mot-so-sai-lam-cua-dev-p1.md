---
layout: post
title: Một số sai lầm của dev P1
---

Hôm nay tôi muốn chia sẻ với các devs một số sai lầm tôi đã gặp phải hoặc được chứng kiến trong công việc. Các chia sẻ mang tính chất chủ quan, không đối địch, ném đá một ai nhé!😀. Hi vọng sẽ được mọi người đón đọc và tìm ra các giải pháp tốt nhất.

## Không “confirm” khi xin nghỉ

Ví dụ bạn có việc đột xuất và bạn xin nghỉ, tuy nhiên bạn không thông báo tới người quản lý dự án, mà chỉ báo cho bên nhân sự. Rắc rối là người quản lý của bạn không hề biết thông tin, nên không có kế hoạch để bố trí nhân lực cho dự án, có thể sẽ chậm deadline, không kịp gửi báo cáo cho khách hàng, hoặc task bạn đang làm cần chuyển giao ngay hôm đó.

### Xin nghỉ sai cách

Bạn có việc đột xuất và cần xin nghỉ, tất nhiên bạn phải vậy. Tuy nhiên nếu bạn chỉ thông báo là “Hôm nay em có việc đột xuất nên xin nghỉ ạ” thì điều đó không tốt lắm.

Sẽ tốt hơn nếu bạn:
– Xác định công việc gấp đó cần hoàn thành trong bao lâu, bạn có thể thông báo cụ thể hơn, em xin nghỉ buổi sáng, hoặc buổi chiều, hay là nghỉ từ mấy giờ đến mấy giờ.
– Tùy vào công việc đột xuất của bạn có riêng tư quá không, nếu không bạn có thể chia sẻ thêm thông tin vì sao xin nghỉ, từ đó người quản lý cũng dễ dàng nhận ra lý do chính đáng và đồng ý cho bạn nghỉ, đồng thời bố trí người khác làm giúp bạn.
– Khi có một kế hoạch xin nghỉ mà bạn xác định từ trước, bạn nên xin phép thông qua email, sớm có thể 1 ngày đối với nghỉ 1 ngày, nghỉ dài thì nên xin phép trước ít nhất 1 tuần. Điều đó sẽ giúp người quản lý sắp xếp nhân sự, bố trí công việc. Tốt hơn hẳn độp một cái bạn nghỉ vèo 1 tuần đúng không?

## Không “confirm” với khách hàng, và cấp trên.

Có thể có những dự án, khách hàng, và sếp khá thoải mái, tuy nhiên nếu đặt mình vào vị trí đó, bạn sẽ nghĩ mình nên xin xác nhận từ cấp trên và khách hàng trước khi làm.

Giả sử có có 1 vấn đề, và bạn nghĩ ra một giải pháp mà bạn nghĩ nó hoàn hảo rồi, sau đó bạn làm không cần confirm.
Trong lòng rất hồ hởi, kiểu này sẽ được khen đây! :) nhưng không bạn lại bị mắng, thậm chí phải đập cái giải pháp của bạn đi, trả lại code trước đó. Đã mất công mất sức, lại còn bị mắng đúng không?

Ngoài ra trong một số tình huống, vì kiến thức của bản thân và kinh nghiệm chưa nhiều, nên chúng ta không lường được hậu quả mà cái giải pháp “hoàn hảo mà chúng ta nghĩ ra”. Nó sẽ ảnh hưởng rất lớn tới hệ thống.
Bạn nên confirm với người phụ trách kỹ thuật của dự án trước khi bắt tay vào làm.

## Làm sai requirement

Cũng tâm lý “giải pháp hoàn hảo”, bạn nghĩ thế mới là tốt và bạn làm, tất nhiên có thể nó tốt hơn thật, bạn làm xong nó chạy tốt hơn. Đúng luôn! Nhưng bạn vẫn sẽ bị mắng, thậm chí đập đi hết thành quả của mình.

Hãy luôn lưu ý rằng, luôn luôn phải làm đúng requirement. Nếu chưa hiểu requirement, phải hỏi khách hàng, làm rõ sau đó mới làm. Làm đúng yêu cầu của khách hàng là trước tiên.

## Vậy nên thế nào? Khách hàng đâu phải lúc nào cũng luôn đúng đâu?

Bạn tìm thấy một giải pháp tốt hơn, HÃY chia sẻ nó, gợi ý, phân tích cho khách hàng, hay là làm rõ với LEADER của mình.
Tất nhiên vẫn là nói trước, sau đó nhận được xác nhận:
– OK giải pháp này tốt, cậu hãy làm đi. Lúc này bạn sẽ áp dụng giải pháp của bạn.
– Không, hãy làm theo cái cũ. Cứ thế mà làm theo cái cũ nhé ![;)].

## Phức tạp quá vấn đề, đơn giản quá vấn đề khiến thời gian estimate luôn bị lệch

Khi bạn được giao 1 task, tuy nhiên bạn có thể suy nghĩ cực kỳ phức tạp hoặc quá đơn giản. Lịch sử chứng minh là estimate của bạn luôn bị lệch với giờ làm thực thế.
Tôi nghĩ bạn nên rút kinh nghiệm từ bản thân, những sai lầm nào thường gặp phải.
– Có những điểm mù trong requirement không?
– Đã estimate thời gian viết test chưa?
– UI để quá nhiều thời gian, trong khi function thì để quá ít?
– Logic chưa rõ ràng, bạn nhảy vào làm, sau đó “rầm” một cái, cái này phải làm sao nhỉ?
Thế là thời gian bị kéo dãn ra. … Có quá nhiều lý do, tuy nhiên bạn NÊN rút từ kinh nghiệm bản thân với các task trước. Sau khi estimate có thể tham khảo với BA, Leader của bạn.
Với những cái nhìn khác nhau, họ có thể thấy những điểm bất hợp lý trong thời gian estimate của bạn.

Đừng estimate quá lố thì bị khách hàng bắt giải thích.
Đừng estimate quá ít, lại khổ cho bản thân.
Hãy estimate, dần dần chính xác với giờ thực tế hơn nhé.

### Tự mình ngồi ngâm cứu giải pháp tốn cả ngày không ra.

Bạn gặp một task khó, cảm thấy mọi người rất bận, nên bạn quyết tâm ngồi ngâm cứu giải pháp cho vấn đề đó.
Tinh thần đó rất tốt, tuy nhiên nếu 30 phút đến 1 giờ bạn không tìm ra, bạn nhất định phải hỏi.
Nếu không bạn sẽ tốn cả ngày, có thể vẫn ko tìm ra được giải pháp. Rất tốn thời gian, và công sức.
Khách hàng không trả tiền cho cả núi giờ đó.

Trái ngược với việc này thì việc hỏi quá nhiều cũng là một vấn đề.

## Không chia nhỏ bài toán, luôn cảm thấy bối rối.

Khi bạn nhận được một bài toán, tốt nhất bạn nên “chia để trị”. Hãy tách các bài toán đó thành những bài toán nhỏ hơn.
Khi chia thành các bài toán nhỏ và đơn giải hơn, bạn sẽ dễ dàng tìm ra lời giải hơn.
Leader của bạn cũng dễ dàng hỗ trợ bạn khi cần hơn là lúc bạn hỏi, cả hai bắt đầu ngồi phân tích, đánh giá bài toán.

#### Không xác định rõ “keyword” để tìm kiếm giải pháp, tâm lý ỷ lại, cái gì cũng hỏi

Tâm lý ỉ lại vào group devs, hay chính leader của bạn, bạn cần gì là đưa câu hỏi lên đó và chờ đợi một câu trả lời mà bạn chỉ cần paste vào đó là chạy.
Điều đó không nên một chút nào. Ai cũng có công việc của mình, hơn nữa bạn như vậy sẽ không thể được đánh giá tốt được.

Bạn nên tự mình xác định vấn đề, keyword của vấn đề, tìm kiếm giải pháp trước.
Nếu không thể tìm ra, khoảng 30 phút, lúc này HÃY NÊN HỎI leader và mọi người.

## Kết luận

Bài chia sẻ cũng dài quá rồi, mình sẽ cắt một số sang bài sau.
Hi vọng mọi người sẽ thảo luận, tìm ra giải pháp tốt cho từng vấn đề.
Luôn đạt được hiệu quả cao trong công việc.

**Sources:**

1. [From Trung Kiên :D](https://www.facebook.com/trungkien.phan.35?fref=ts)
