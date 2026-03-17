# Option 1 — Best Practice: Use a Virtual Environment (Recommended)

This keeps packages isolated per project.

sudo apt install python3-venv python3-full -y

# create venv
python3 -m venv venv

# activate
source venv/bin/activate

# install package
pip install requests

You’ll see (venv) in terminal → now it’s safe.

# To exit:
deactivate
