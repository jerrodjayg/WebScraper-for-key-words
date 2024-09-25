import requests
from bs4 import BeautifulSoup
import re
import csv

# List of URLs to search across
urls = [
    "https://example.com/page1",
    "https://example.com/page2",
    # Add more URLs here
]

# Keywords to find
keywords = ['transparency', 'corruption', 'accountability']

# CSV file to store results
output_file = "keyword_search_results.csv"

def fetch_content(url):
    """Fetch the content of a web page."""
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return response.text
        else:
            print(f"Failed to retrieve content from {url}")
            return None
    except Exception as e:
        print(f"Error fetching content from {url}: {e}")
        return None

def find_keywords(content, keywords):
    """Find and return sentences containing the keywords."""
    soup = BeautifulSoup(content, 'html.parser')
    text = soup.get_text()  # Get all text from the web page
    matches = []
    
    # Break text into sentences
    sentences = re.split(r'[.!?]', text)
    
    for sentence in sentences:
        for keyword in keywords:
            if keyword.lower() in sentence.lower():  # Case insensitive search
                matches.append((keyword, sentence.strip()))
    
    return matches

def search_keywords_across_pages(urls, keywords):
    """Search for keywords across multiple web pages."""
    results = []
    
    for url in urls:
        print(f"Searching in {url}...")
        content = fetch_content(url)
        if content:
            matches = find_keywords(content, keywords)
            if matches:
                for keyword, sentence in matches:
                    results.append([url, keyword, sentence])
    
    return results

def save_results_to_csv(results, output_file):
    """Save the search results to a CSV file."""
    with open(output_file, mode='w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(["URL", "Keyword", "Sentence"])
        for result in results:
            writer.writerow(result)

# Execute the search and save results
results = search_keywords_across_pages(urls, keywords)

if results:
    save_results_to_csv(results, output_file)
    print(f"Results have been saved to {output_file}")
else:
    print("No matches found.")

