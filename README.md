# DesafioPoc
import sqlite3

from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
from sqlalchemy.dialects import sqlite

# Configurando o webdriver do Chrome
options = webdriver.ChromeOptions()
driver = webdriver.Chrome(options=options)
url = 'https://www.vriconsulting.com.br/guias/guiasIndex.php?idGuia=22'
driver.get(url)
driver.implicitly_wait(10)

# Encontrar a tabela
table_xpath = '/html/body/section/section[2]/table[1]'
table = driver.find_element(by=By.XPATH, value=table_xpath)

html_content = table.get_attribute('outerHTML')
soup = BeautifulSoup(html_content, 'html.parser')

db_path = 'dados.db'
conn = sqlite3.connect(db_path)
cursor = conn.cursor()


cursor.execute('''DROP TABLE tabela''')

cursor.execute('''
    CREATE TABLE IF NOT EXISTS tabela (
        N TEXT, 
        Campo TEXT,
        Descricao TEXT,
        Tipo TEXT,
        Tam TEXT,
        Dec TEXT,
        Entr TEXT,
        Saida TEXT
    )
''')

rows = soup.find('tbody').find_all('tr')
for row in rows:
        columns = row.find_all('td')
        data = [col.get_text(strip=True) for col in columns]

        cursor.execute('''
            INSERT INTO tabela (N, Campo, Descricao, Tipo, Tam, Dec, Entr, Saida)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?)
        ''', data)

conn.commit()
conn.close()

driver.quit()
