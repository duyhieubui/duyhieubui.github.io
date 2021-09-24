---
title: 'Google Skywater 130nm MPW-3 Quickstart'
date: 2021-09-23
permalink: /posts/2021/09/Google-Skywater-130nm-MPW-3/
tags:
  - Skywater 130nm
  - Opensource PDK
  - Hardware design
---
## Giới thiệu

Bài viết này sẽ trình bày cách cài đặt tự động các công cụ mã nguồn mở
để sử dụng với Google Skywater 130nm Opensource PDK. Và cài đặt docker
với bộ công cụ OpenLane/OpenRoad để thực thi các thiết kế phần cứng
số. 

Các công cụ được cài đặt bao gồm:
* Xschem: Công cụ để vẽ mạch điện
* NGSpice: Công cụ mô phỏng spice 
* Magic: Công cụ layout, tríc xuất phần tử ký sinh (RC Extraction),
  kiểm tra luật thiết kế (Design Rule Check - DRC)
* Klayout: Công cụ layout, kiểm tra luật thiết kế (Design Rule Check)
* Netgen: Công cụ thực hiện kiểm tra sự tương đồng giữa layout và mạch
  điện (Layout versus Schematic - LVS)
* GAW: chương trình hiển thị dạng sóng sau khi mô phỏng spice bằng NGSpice.
* GTKWave: Chương trình hiện thị dạng sóng cho định dạng VCD. File
  .vcd có thể được trích xuất trong qua trình mô phỏng Verilog/VHDL sử
  dụng các công cụ mô phỏng mã nguồn mở.
* iverilog: Chương trình mô phỏng Verilog
* Openlane docker image: Đây là container chứa bộ công cụ dùng cho
  openlane. Nó bao gồm các công cụ trên cùng với phần mềm
  OpenRoad/OpenLane. Việc cài đặt các công cụ này vào máy tính không
  dùng docker container sẽ được trình bày trong một bài khác.

Chú ý: Những công cụ trên, các bạn có thể cài đặt một số công cụ bằng
lệnh 'apt-get' trong Ubuntu. Tuy nhiên, các phần mềm này quá cũ và có
thể chưa được hỗ trợ các tính năng mới của open\_pdks và Google Skywater
130nm Opensource PDK. Do vậy, phần lớn các phần mềm được đề cập ở đây
được cài đặt từ mã nguồn.

Để cài đặt các phần mềm này, các bạn có thể sử dụng [script tại
đây](https://gist.github.com/duyhieubui/3e2703f5980f34c2285e9ea0c8bbdd4a)

## Các bước cài đặt bằng dòng lệnh trong Ubuntu 20.04 hoặc Windows Subsystem for Linux 2 (WSL2)

Đối với người dùng windows, các bạn cần cài đặt Windows Subsystem for
Linux 2 (WSL2), cài đặt Ubuntu 20.04 cho WSL2, và chạy GUI dùng VcXsrv
theo hướng dẫn [tại
đây](https://medium.com/@japheth.yates/the-complete-wsl2-gui-setup-2582828f4577). Sau
khi đã cài đặt thành công, các bạn có thể làm theo hướng dẫn bên dưới.


Chú ý:
* Dấu $ ở đầu dòng có nghĩa là lệnh này phải được chạy ở trong giao
  diện dòng lệnh (terminal) của linux.

Kiểm tra xem các bạn đã có chương trình wget hay chưa bằng lệnh sau:

<pre>
$ which wget
</pre>

Nếu chương trình wget đã được cài đặt, nó sẽ trả về như sau:

<pre>
$ /usr/bin/wget
</pre>

Nếu chương trình không trả về gì cả, có nghĩa là bạn chưa có chương
trình wget và bạn có thể cài đặt nó bằng lệnh:

<pre>
$ sudo apt-get install wget
</pre>

Sau đó, các công cụ có thể được cài đặt dùng các lệnh sau:

<pre>
$ wget "https://gist.githubusercontent.com/duyhieubui/3e2703f5980f34c2285e9ea0c8bbdd4a/raw/0c44075931dd2b1370f1f226edc8df4222e09ece/build-tool-ubuntu.sh"
$ bash build-tool-ubuntu.sh
</pre>

Nếu các bạn bị hỏi password, thì các bạn nhập password của mình vào
rồi ấn Enter.

Chú ý: Mặc định khi gõ password trong terminal của linux, không có ký
tự nào được hiển thị cả, nên bạn cứ gõ đúng password và ấn "Enter".

Sau khi chương trình chạy xong, các phần mềm đuợc biên dịch từ mã nguồn sẽ
được cài đặt vào thư mục $HOME/efabless/tools. Để kiểm tra quá trình
build có lỗi gì hay không các bạn có thể sử dụng lệnh sau:

<pre>
$ grep -i '^error:' $HOME/efabless/build/run.log
</pre>

* Trường hợp không lỗi, lệnh trên sẽ không trả về gì.

* Trường hợp có lỗi, lệnh trên sẽ trả về lỗi gặp phải trong quá trình
biên dịch và chạy script.

Để chạy lại quá trình build, các bạn phải xoá thư mục $HOME/efabless/build.

Sau khi cài đặt xong, người dùng cần phải đăng xuất (log out) và đăng
nhập lại để cập nhật lại người dùng vào nhóm docker. Nếu không thực
hiện bước này, bạn có thể gặp lỗi ko có quyền (permission) để chạy
docker.

## Cài đặt PDK và Openlane sử dụng mã nguồn của Efabless

Để tham gia vào chương trình MPW-3, các bạn sẽ phải đưa thiết kế của
mình vào caravel\_user\_project. Sau đó, thiết kế này sẽ đưọc ghép với
Caravel Test Harness để gửi đi chế tạo. Do vậy, trong bài này, tôi sẽ
hướng dẫn các bạn cài đặt PDK sử dụng luôn mã nguồn của Efabless trong
caravel\_user\_project.

### Thiết lập môi trường với các phần mềm đã cài đặt ở phần trước
Để thiết lập môi trường chạy các phần mềm đã được cài đặt ở phần
trước, các bạn chỉ cần chạy lệnh sau trong cửa sổ dòng lệnh (terminal)
mà các bạn sẽ chạy chương trình:

<pre>
$ source $HOME/efabless/env.sh
</pre>

Chú ý: Nếu các bạn mở 1 terminal mới, các bạn cũng cần chạy lại lệnh này.

### Tải về mã nguồn của caravel\_user\_project và tiến hành cài đặt

Các bạn có thể tải về mã nguồn của caravel\_user\_project từ github của dự án như sau:

<pre>
$ cd $HOME/efabless
$ git clone https://github.com/efabless/caravel_user_project
</pre>

Nếu các bạn đã fork dự án này ra thành project của cá nhân, thì các
bạn có thể dùng đường dẫn đến dự án của các bạn.

<pre> $ git clone url-đến-project </pre>

Chú ý: Các bạn nên đảm bảo đã dùng đúng mã nguồn cho MPW-3. Nó có tag là mpw-3.

Sau đó các bạn di chuyển đến thư mục vừa tải về và thiết lập biến môi
trường trước khi đặt các cài đặt như sau:

<pre>
$ export PDK_ROOT=$HOME/efabless/pdks
$ export OPENLANE_TAG=2021.09.19_20.25.16
$ export OPENLANE_ROOT=$HOME/efabless/openlane/$OPENLANE_TAG
$ export CARAVEL_ROOT=$PWD/caravel
</pre>

Chú ý:
* Đối với MPW-3, version mới nhất của openlane có bug (tôi đã test
  ngày 23/9/2021). Do vậy, tạm thời các bạn nên sử dụng openlane
  version như ở trên.
  
Các lệnh thiết lập biến môi trường này cũng có thể đưọc lưu thành file
để sử dụng lại.  Ví dụ tôi sẽ lưu các cấu hình của dự án trong file
setup.sh trong thư mục của dự án. Nội dung của file này như sau:

<pre>
$ source $HOME/efabless/env.sh
$ export PDK_ROOT=$HOME/efabless/pdks
$ export OPENLANE_TAG=2021.09.19_20.25.16
$ export OPENLANE_ROOT=$HOME/efabless/openlane/$OPENLANE_TAG
$ export CARAVEL_ROOT=$PWD/caravel
</pre>

Mỗi khi mở một cửa sổ dòng lệnh (terminal) mới, các bạn có thể dùng
lệnh cd để di chuyển đến thư mục của dự án và áp dụng các cấu hình
này. Ví dụ:

<pre>
$ cd $HOME/efabless/caravel_user_project
$ source setup.sh
</pre>

Tải mã nguồn của caravel-lite, một phần quan trọng để chạy mô phỏng và
thực thi phần cứng cho user_project bằng lệnh:

<pre>
$ make install
$ make pdk
$ make openlane
</pre>

Nếu các lệnh này không báo lỗi, các bạn đã cài đặt pdk thành công và
sẵn sàng để chạy ví dụ của dự án caravel\_user\_project.

## Chạy thử ví dụ trong caravel\_user\_project

Trong dự án này, đã có sẵn thiết kế của một bộ đếm. Các script tự động
sẽ tự động tổng hợp phần cứng, chạy định tuyến và đặt chỗ (place &
route), DRC & LVS. Các script tự động này được định nghĩa trong
OpenLane và Openroad. Người dùng chỉ cần cung cấp file cấu hình
(config.tcl), các phần mềm này sẽ tự động thực thi phần cứng dùng công
nghệ 130nm. Tác động vào các phần mềm này nằm ngoài phạm vi của bài
viết này. Các chủ đề này có thể được trình bày trong các bài viết
khác.

Trước khi chạy các script tự động, bước đầu tiên là chúng ta cần thiết
lập các biến môi trường như sau:

<pre>
$ source $HOME/efabless/env.sh
$ export PDK_ROOT=$HOME/efabless/pdks
$ export OPENLANE_TAG=2021.09.19_20.25.16
$ export OPENLANE_ROOT=$HOME/efabless/openlane/$OPENLANE_TAG
$ export CARAVEL_ROOT=$PWD/caravel
</pre>

Nếu các bạn đã tạo file setup.sh thì các bạn chỉ cần chạy lệnh sau:
<pre>
$ source setup.sh
</pre>

### Chạy mô phỏng RTL

Mặc định mã nguồn RTL của dự án được lưu ở thư mục 'verilog/rtl'. Các
bạn có thể vào thư mục này để tham khảo file thiết kế
user\_proj\_example.v và file user\_project\_wrapper.v.

Các kịch bản kiểm tra được đặt trong thư mục 'verilog/dv'.

Để chạy mô phỏng, các bạn có thể download docker image chứa các công cụ chạy mô phỏng bằng lệnh:

<pre>
$ make simenv
</pre>

Để chạy mô phỏng một kịch bản kiểm tra, các bạn có thể chạy lệnh như sau:
<pre>
$ make verify-<tên thư mục chứa kịch bản kiểm tra trong thư mục verilog/dv>
</pre>

Ví dụ:

<pre>
$ make verify-io_ports
</pre>

Kết quả sau khi chạy mô phỏng thành công:
<pre>
MPRJ-IO state = 11101011 
MPRJ-IO state = 11101100 
MPRJ-IO state = 11101101 
MPRJ-IO state = 11101110 
MPRJ-IO state = 11101111 
MPRJ-IO state = 11110000 
MPRJ-IO state = 11110001 
MPRJ-IO state = 11110010 
MPRJ-IO state = 11110011 
MPRJ-IO state = 11110100 
MPRJ-IO state = 11110101 
MPRJ-IO state = 11110110 
MPRJ-IO state = 11110111 
MPRJ-IO state = 11111000 
MPRJ-IO state = 11111001 
MPRJ-IO state = 11111010 
MPRJ-IO state = 11111011 
MPRJ-IO state = 11111100 
MPRJ-IO state = 11111101 
MPRJ-IO state = 11111110 
MPRJ-IO state = 11111111 
Monitor: Test 1 Mega-Project IO (RTL) Passed
rm io_ports.elf io_ports.vvp
</pre>

### Chạy thực thi phần cứng

Các mã nguồn cấu hình để thực thi phần cứng được đặt trong thư mục
'openlane/<tên-thiết-kế>'. Trong đó, file config.tcl là file config
chính của openlane. Các bạn có thể tạo một thiết kết mới bằng cách tạo 1 thư mục với
file config.tcl trong thư mục 'openlane'.

Sau khi đã có file cấu hình, quá trình thực thi phần cứng được thực hiện bằng lệnh:

<pre>
$ make tên-thiết-kế
</pre>

Mặc định, caravel\_user\_project đã có 2 thiết kế là user\_proj\_example và user\_project\_wrapper.

Chạy thực thi phần cứng thiết kế user\_proj\_example:

<pre>
$ make user_proj_example
</pre>

Lệnh này sẽ thực thi ví dụ có cấu hình trong thư mục
'openlane/user\_proj\_example'. Nếu thực thi thành công, file gds chứa
bản vẽ của thiết kế sẽ được tạo trong thư mục gds.

Kết quả sau khi chạy:
<pre>
[INFO]: Saving Magic Views in /project
[INFO]: Calculating Runtime From the Start...
[INFO]: flow completed for user_proj_example/24-09_02-32 in 0h8m8s
[INFO]: Saving Runtime Environment
[INFO]: Generating Final Summary Report...
[INFO]: Design Name: user_proj_example
Run Directory: /project/openlane/user_proj_example/runs/user_proj_example
----------------------------------------

Magic DRC Summary:
Source: /project/openlane/user_proj_example/runs/user_proj_example/reports/magic/38-magic.drc
Total Magic DRC violations is 0
----------------------------------------

LVS Summary:
Source: /project/openlane/user_proj_example/runs/user_proj_example/results/lvs/user_proj_example.lvs_parsed.lef.log
LVS reports no net, device, pin, or property mismatches.
Total errors = 0
----------------------------------------

Antenna Summary:
Source: /project/openlane/user_proj_example/runs/user_proj_example/reports/routing/40-antenna.rpt
Number of pins violated: 4
Number of nets violated: 4
[INFO]: check full report here: /project/openlane/user_proj_example/runs/user_proj_example/reports/final_summary_report.csv
</pre>

Các file báo cáo trong quá trình thực hiện và kết quả tức thời được
tạo ra trong thư mục
'openlane/user\_proj\_example/runs/user\_proj\_example'.

Xem layout của thiết kế:
<pre>
$ klayout openlane/user_proj_example/runs/user_proj_example/results/routing/23-user_proj_example.def
</pre>

Xem dạng GDS của thiết kế:
<pre>
$ klayout gds/user_proj_example.gds
</pre>

Để tích hợp user\_project\_example vào user\_project\_wrapper,
các bạn có thể thực thi lệnh này:

<pre>
$ make user_project_wrapper
</pre>

Kết quả sau khi chạy thành công:
<pre>
[INFO]: current step index: 40
[INFO]: Your design contains macros, which is not supported by the current integration of CVC. So CVC won't run, however CVC is just a check so it's not critical to your design.
[INFO]: Saving Magic Views in /project
[INFO]: Calculating Runtime From the Start...
[INFO]: flow completed for user_project_wrapper/24-09_03-02 in 0h5m51s
[INFO]: Saving Runtime Environment
[INFO]: Generating Final Summary Report...
[INFO]: Design Name: user_project_wrapper
Run Directory: /project/openlane/user_project_wrapper/runs/user_project_wrapper
----------------------------------------

Magic DRC Summary:
Source: /project/openlane/user_project_wrapper/runs/user_project_wrapper/reports/magic/38-magic.drc
Total Magic DRC violations is 0
----------------------------------------

LVS Summary:
Source: /project/openlane/user_project_wrapper/runs/user_project_wrapper/results/lvs/user_project_wrapper.lvs_parsed.lef.log
LVS reports no net, device, pin, or property mismatches.
Total errors = 0
----------------------------------------

Antenna Summary:
Source: /project/openlane/user_project_wrapper/runs/user_project_wrapper/reports/routing/40-antenna.rpt
Number of pins violated: 0
Number of nets violated: 0
[INFO]: check full report here: /project/openlane/user_project_wrapper/runs/user_project_wrapper/reports/final_summary_report.csv
[SUCCESS]: Flow Completed Without Fatal Errors.
mkdir -p ../signoff/user_project_wrapper/
cp user_project_wrapper/runs/user_project_wrapper/OPENLANE_VERSION ../signoff/user_project_wrapper/
cp user_project_wrapper/runs/user_project_wrapper/PDK_SOURCES ../signoff/user_project_wrapper/
cp user_project_wrapper/runs/user_project_wrapper/reports/final_summary_report.csv ../signoff/user_project_wrapper/
make[1]: Leaving directory '/home/dhb/efabless/caravel_user_project/openlane'
</pre>

Xem layout của thiết kế:
<pre>
$ klayout openlane/user_proj_example/runs/user_proj_example/results/routing/23-user_proj_example.def
</pre>

Xem dạng GDS của thiết kế:
<pre>
$ klayout gds/user_project_wrapper.gds
</pre>

Thiết kế này chỉ nối dây từ user\_proj\_example ra các chân IO của
chip. Do vậy, các bạn sẽ chỉ nhìn thấy các dây dẫn được nối.

Voila! Như vậy là chúng ta đã hoàn thành cài đặt công cụ, cài đặt pdk
và chạy thử 1 project ví dụ. Các bạn có thể thử đưa thiết kế của mình
vào caravel_user_project để gửi đi chế tạo trong MPW-3 của Google.


Nếu các bạn có câu hỏi hoặc góp ý/đóng góp cho bài viết. Các bạn có
thể để lại comment hoặc reaction trong phần bên dưới. Các bạn cần
login account github để có thể gửi comment.

Cảm ơn các bạn đã đọc bài và chúc các bạn thành công!
