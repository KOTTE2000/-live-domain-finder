# live-domain-finder

import subprocess
import sys
import os

# Check for tool installation and install if necessary
def install_tool(tool_name, install_command):
    print(f"Checking for {tool_name}...")
    try:
        subprocess.run([tool_name, "--version"], capture_output=True, check=True)
        print(f"{tool_name} is already installed.")
    except subprocess.CalledProcessError:
        print(f"{tool_name} not found. Installing...")
        subprocess.run(install_command, shell=True, check=True)
        print(f"{tool_name} installed.")

def main():
    # Define domain and files
    domain = input("Enter the main domain to scan: ")
    subdomains_file = "subdomains.txt"
    live_httpx_file = "live_httpx.txt"
    live_httprobe_file = "live_httprobe.txt"

    # Prerequisite installation commands
    install_tool("subfinder", "go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest")
    install_tool("httpx", "go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest")
    install_tool("httprobe", "go get -u github.com/tomnomnom/httprobe")

    # Run subfinder to find subdomains
    print("Running subfinder...")
    subfinder_result = subprocess.run(
        ["subfinder", "-d", domain, "-silent"],
        capture_output=True,
        text=True
    )

    subdomains = subfinder_result.stdout.split()
    print(f"Found {len(subdomains)} subdomains")

    # Save subdomains to a file
    with open(subdomains_file, "w") as file:
        for subdomain in subdomains:
            file.write(f"{subdomain}\n")

    # Run httpx to check HTTP status of subdomains
    print("Running httpx...")
    httpx_result = subprocess.run(
        ["httpx", "-l", subdomains_file, "-silent"],
        capture_output=True,
        text=True
    )

    live_httpx = httpx_result.stdout.split()
    print(f"Found {len(live_httpx)} live HTTP services with httpx")

    # Save live httpx results to a file
    with open(live_httpx_file, "w") as file:
        for line in live_httpx:
            file.write(f"{line}\n")

    # Run httprobe to identify live HTTP services
    print("Running httprobe...")
    httprobe_result = subprocess.run(
        ["cat", subdomains_file, "|", "httprobe", "-c", "50", "-t", "3000"],
        shell=True,
        capture_output=True,
        text=True
    )

    live_httprobe = httprobe_result.stdout.split()
    print(f"Found {len(live_httprobe)} live HTTP services with httprobe")

    # Save live httprobe results to a file
    with open(live_httprobe_file, "w") as file:
        for line in live_httprobe:
            file.write(f"{line}\n")

    print("Process completed!")

if _name_ == "_main_":
    if not sys.warnoptions:
        import warnings
        warnings.simplefilter("ignore")
    main()
