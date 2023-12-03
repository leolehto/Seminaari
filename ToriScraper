import requests
from bs4 import BeautifulSoup as BS

def translate_condition_to_number(condition):
    condition_mapping = {
        "Uusi": 5,
        "Erinomainen": 4,
        "Hyvä": 3,
        "Tyydyttävä": 2,
        "Huono": 1
    }
    return condition_mapping.get(condition, 0)
def cond(link):
    url = link

    page = requests.get(url)
    soup = BS(page.content, 'html.parser')

    cond_value = None

    # Find the "Kunto:" topic and its value in the table
    topic_elem = soup.find('td', class_='topic', string='Kunto:')
    if topic_elem:
        value_elem = topic_elem.find_next('td', class_='value')
        if value_elem:
            cond_value = value_elem.get_text().strip()

    return(cond_value)

def ScrapeTori(num_pages=2):
    base_url = 'https://www.tori.fi/koko_suomi/sisustus_ja_huonekalut?ca=18&q=sisustus%20ja%20huonekalut&cg=3020&st=s&w=3&o={}'

    scraped_data = []

    for page_number in range(1, num_pages + 1):
        url = base_url.format(page_number)
        response = requests.get(url)

        if response.status_code != 200:
            print(f"Failed to retrieve page {page_number}, status code: {response.status_code}")
            continue

        page = BS(response.content, 'html.parser')
        products = page.find_all('a', class_='item_row_flex')

        if not products:
            print(f"No items found on page {page_number}")
            continue

        for i, product in enumerate(products, start=1):
            try:
                title = product.find('div', class_="li-title").contents[0]
                price = product.find('p', class_="list_price ineuros").contents[0].replace(" ", "")
                product_url = product.get('href')

                cleaned_price = ''.join(p for p in price if p.isdigit())
                price_int = int(cleaned_price)

                image_container = product.find('div', class_='image_container')
                img_url = 'N/A'
                if image_container:
                    image_div = image_container.find('div', class_='item_image_div')
                    img_class = image_div.find('img', class_='item_image') if image_div else None
                    img_url = img_class['src'] if img_class and 'src' in img_class.attrs else 'N/A'

                # Get the product condition
                condition = cond(product_url)
                condition_number = translate_condition_to_number(condition)

                # Create a dictionary for each item
                item_data = {
                    "Title": title,
                    "Price": price_int,
                    "Image URL": img_url,
                    "Condition": condition_number
                }

                scraped_data.append(item_data)

                print(f"Page {page_number}, Item {i} - Title: {item_data['Title']}, Price: {item_data['Price']}, Image URL: {item_data['Image URL']}, Condition: {item_data['Condition']}")

            except Exception as e:
                print(f"Error processing item {i} on page {page_number}: {e}")

        print(f'Page {page_number} - listings: {len(scraped_data)}')

    return scraped_data

if __name__ == "__main__":
    ScrapeTori()
