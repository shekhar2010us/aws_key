# Run Selenium Test in Docker Containers


### Pre-requisites
1. Docker installed
2. Download the ChromeDriver (http://chromedriver.chromium.org/downloads)
3. Pull `selenium/standalone-chrome` image
4. Run a container
5. Bind it to Language like Python, Ruby, etc.





```python
cd ~
mkdir selenium_test
cd selenium_test/

docker pull selenium/standalone-chrome

docker run -d -v /home/ubuntu/selenium_test:/home/seluser/Downloads -p 4445:4444 --name selenium_container selenium/standalone-chrome

```
<br><br>

### 1. Create a python file, `test_purchase_tickets.py`

```
# coding=utf-8
import helpers.helpers as utils
import pytest
from selenium.webdriver.common.by import By


def submit_form(driver):
    driver.find_element(By.XPATH, "//input[@type='submit']").click()


def find_flight_in_dropdown(dropdown_element, expected_flight):
    for option in dropdown_element.find_elements_by_tag_name('option'):
        if option.text == expected_flight:
            option.click()


def choose_departure_flight(driver, departure_flight):
    from_port_dropdown = utils.find_element(driver, By.NAME, "fromPort")
    find_flight_in_dropdown(dropdown_element=from_port_dropdown, expected_flight=departure_flight)


def choose_arrival_flight(driver, arrival_flight):
    to_port_dropdown = utils.find_element(driver, By.NAME, "toPort")
    find_flight_in_dropdown(dropdown_element=to_port_dropdown, expected_flight=arrival_flight)


@pytest.fixture(scope="function")
def open_blazedemo(driver, url):
    driver.get(url)
    driver.find_element(By.TAG_NAME, "form")
    assert driver.title == "BlazeDemo"


@pytest.mark.parametrize('from_port, to_port', [
    ("Paris", "Buenos Aires"),
    ("Philadelphia", "Rome"),
    ("Boston", "London"),
    ("Portland", "Berlin"),
    ("San Diego", "New York"),
    ("Mexico City", "Dublin"),
    # fail on purpose to verify screenshot was added to allure report
    ("Tel Aviv", "Dubai")
])
def test_find_flights(driver, open_blazedemo, from_port, to_port):

    # Find flight
    choose_departure_flight(driver, departure_flight=from_port)
    choose_arrival_flight(driver, arrival_flight=to_port)
    submit_form(driver)

    assert from_port in utils.get_text(driver, By.TAG_NAME, "h3"), "{} was not found".format(from_port)
    assert to_port in utils.get_text(driver, By.TAG_NAME, "h3"), "{} was not found".format(to_port)
    assert "reserve.php" in driver.current_url

    # Choose flight
    submit_form(driver)

    assert from_port in utils.get_text(driver, By.TAG_NAME, "h2"), "{} was not found".format(from_port)
    assert to_port in utils.get_text(driver, By.TAG_NAME, "h2"), "{} was not found".format(to_port)
    assert "purchase.php" in driver.current_url

    # Purchase flight
    submit_form(driver)

    assert "Thank you for your purchase today!" in utils.get_text(driver, By.TAG_NAME, "h1")
    assert "confirmation.php" in driver.current_url


def test_teardown(driver):
    driver.quit()
```

### 2. Create `requirements.txt` file

```
arrow==0.8.0
colorama==0.3.7
enum34==1.1.6
lxml==3.6.0
moment==0.5.1
namedlist==1.7
py==1.4.31
pytest==3.5.0
pytest-allure-adaptor==1.7.10
pytest-gitignore==1.3
python-dateutil==2.5.3
pytz==2016.4
selenium==2.53.5
six==1.10.0
times==0.7
```

### 3. Create `Dockerfile`

```
FROM python:2.7-stretch

RUN apt-get update && apt-get install -yq \
    firefox-esr=52.6.0esr-1~deb9u1 \
    chromium=62.0.3202.89-1~deb9u1 \
    git-core=1:2.11.0-3+deb9u2 \
    xvfb=2:1.19.2-1+deb9u2 \
    xsel=1.2.0-2+b1 \
    unzip=6.0-21 \
    python-pytest=3.0.6-1 \
    libgconf2-4=3.2.6-4+b1 \
    libncurses5=6.0+20161126-1+deb9u2 \
    libxml2-dev=2.9.4+dfsg1-2.2+deb9u2 \
    libxslt-dev \
    libz-dev \
    xclip=0.12+svn84-4+b1

# GeckoDriver v0.19.1
RUN wget -q "https://github.com/mozilla/geckodriver/releases/download/v0.19.1/geckodriver-v0.19.1-linux64.tar.gz" -O /tmp/geckodriver.tgz \
    && tar zxf /tmp/geckodriver.tgz -C /usr/bin/ \
    && rm /tmp/geckodriver.tgz

# chromeDriver v2.35
RUN wget -q "https://chromedriver.storage.googleapis.com/2.35/chromedriver_linux64.zip" -O /tmp/chromedriver.zip \
    && unzip /tmp/chromedriver.zip -d /usr/bin/ \
    && rm /tmp/chromedriver.zip

# xvfb - X server display
ADD selenium-base-image/xvfb-chromium /usr/bin/xvfb-chromium
RUN ln -s /usr/bin/xvfb-chromium /usr/bin/google-chrome \
    && chmod 777 /usr/bin/xvfb-chromium

# create symlinks to chromedriver and geckodriver (to the PATH)
RUN ln -s /usr/bin/geckodriver /usr/bin/chromium-browser \
    && chmod 777 /usr/bin/geckodriver \
    && chmod 777 /usr/bin/chromium-browser
```



https://www.freecodecamp.org/news/a-recipe-for-website-automated-tests-with-python-selenium-headless-chrome-in-docker-8d344a97afb5/



