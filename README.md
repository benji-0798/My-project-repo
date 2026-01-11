import argparse
import requests
import sys

GITHUB_API = "https://api.github.com"


def get_headers(token):
    if token:
        return {"Authorization": f"token {token}"}
    return {}


def get_repo_info(owner, repo, headers):
    url = f"{GITHUB_API}/repos/{owner}/{repo}"
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()


def get_languages(owner, repo, headers):
    url = f"{GITHUB_API}/repos/{owner}/{repo}/languages"
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()


def get_contributors(owner, repo, headers, top_n=5):
    url = f"{GITHUB_API}/repos/{owner}/{repo}/contributors"
    response = requests.get(url, headers=headers)
    response.raise_for_status()
    return response.json()[:top_n]


def analyze_repo(owner, repo, token):
    headers = get_headers(token)

    try:
        repo_info = get_repo_info(owner, repo, headers)
        languages = get_languages(owner, repo, headers)
        contributors = get_contributors(owner, repo, headers)

    except requests.exceptions.HTTPError as e:
        print(" Error:", e)
        sys.exit(1)

    print("\n Repository Information")
    print("-" * 40)
    print(f"Name : {repo_info['full_name']}")
    print(f"Description : {repo_info['description']}")
    print(f"Stars : {repo_info['stargazers_count']}")
    print(f"Forks : {repo_info['forks_count']}")
    print(f"Open Issues : {repo_info['open_issues_count']}")
    print(f"License : {repo_info['license']['name'] if repo_info['license'] else 'None'}")

    print("\n Languages Used")
    print("-" * 40)
    total_bytes = sum(languages.values())
    for lang, size in languages.items():
        percent = (size / total_bytes) * 100
        print(f"{lang:<15} {percent:.2f}%")

    print("\n Top Contributors")
    print("-" * 40)
    for c in contributors:
        print(f"{c['login']:<20} Contributions: {c['contributions']}")

    print("\n Analysis Complete\n")


def main():
    parser = argparse.ArgumentParser(description="CLI GitHub Repository Analyzer")
    parser.add_argument("repo", help="Repository in format owner/repo")
    parser.add_argument("--token", help="GitHub Personal Access Token", default=None)

    args = parser.parse_args()

    if "/" not in args.repo:
        print(" Repo format must be owner/repo")
        sys.exit(1)

    owner, repo = args.repo.split("/")
    analyze_repo(owner, repo, args.token)


if __name__ == "__main__":
    main()
