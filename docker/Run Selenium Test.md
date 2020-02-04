# Run Selenium in Docker Containers


### Pre-requisites

##### We will run Selenium in a standalone Ubuntu image
1. Assuming - Docker installed
2. Download the ChromeDriver (http://chromedriver.chromium.org/downloads)
3. Pull `ubuntu:latest` image
4. Write selenium test in Python
5. Build the image
5. Run the container


### 1. Setup directory and pull image
```
cd ~
mkdir selenium_test
cd selenium_test/

docker pull ubuntu:latest
```
<br>

### 2. Create a python file, `app.py`

```
from time import sleep
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
driver = webdriver.Chrome('/chromedriver' ,chrome_options=chrome_options)

print ('Getting Selenium Chrome WebDriver..')
driver.get('http://www.ubuntu.com/')
print ('Sleeping for 3 seconds')
sleep(3.0)

if 'ubuntu' in driver.title.lower():
    print ('ubuntu is in title of www.ubuntu.com - TEST PASSED')
else:
    print ('ubuntu is not in the title of www.ubuntu.com - TEST FAILED')
driver.quit()

print ('Exiting....')
```


### 3. Create `Dockerfile`

```
FROM ubuntu:latest

RUN apt-get update -y
RUN apt-get install -y chromium-chromedriver python3.6 python3-pip python-selenium python3-selenium vim wget tree
RUN apt-get install -y zip

RUN wget https://chromedriver.storage.googleapis.com/79.0.3945.36/chromedriver_linux64.zip
RUN unzip chromedriver_linux64.zip

COPY app.py .

CMD ["python3", "app.py"]
```

### 4. Build image

```
docker build -t myselenium:1.0 .
```

### 5. Run Container

```
docker run --name sel myselenium:1.0
```


## Alternatively
You can use the image `selenium/standalone-chrome` that comes with browser and the chromedriver

