Initial Project Structure
github-repo-analyzer/
├── analyzer.py
├── cli.py
├── utils.py
├── requirements.txt
└── README.md



# My-project-repoimport argparse
from analyzer import analyze_repo

def main():
    parser = argparse.ArgumentParser(description="GitHub Repo Analyzer CLI Tool")
    parser.add_argument("repo", help="GitHub repository in the format owner/repo")
    parser.add_argument("--token", help="GitHub personal access token (optional)", default=None)
    args = parser.parse_args()

    analyze_repo(args.repo, args.token)

if __name__ == "__main__":
    main()
    import requests
from utils import print_repo_info

def analyze_repo(repo_name, token=None):
    headers = {"Authorization": f"token {token}"} if token else {}
    url = f"https://api.github.com/repos/{repo_name}"

    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        repo_data = response.json()
        print_repo_info(repo_data)
    else:
        print(f"Failed to fetch repo data: {response.status_code} - {response.text}")
