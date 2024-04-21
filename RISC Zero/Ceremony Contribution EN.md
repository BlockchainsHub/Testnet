![RISC Zero Banner](https://github.com/BlockchainsHub/Testnet/assets/77204008/ee01db53-478b-4eb6-85ce-1bc532d46825)

# RISC Zero Ceremony Contribution Guide
This guide will help you to participate in the RISC Zero Ceremony Contribution.

-----------------------------------------------------------

## System Requirements
| System | Requirements |
|-|-
| OS | Linux |
| Memory | 8 GB |
| Bandwidth | At least 25 Mbps |

-----------------------------------------------------------

## Github Requirements
Before continuing make sure your GitHub account meets this requirement.
- Following 5 people
- Followed by 1 
- Have 2 public repository
- At least one month old

-----------------------------------------------------------

## Installing Dependencies
### Installing Node.js
Install NVM (Node Version Manager)
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Download and install Node.js
```bash
nvm install 20
```

Verifies the right Node.js version is in the environment. It should show `v20.12.2`
```bash
node -v
```

Verifies the right NPM version is in the environment. It should show `10.5.0`
```bash
npm -v
```

### Installing Tmux
```bash
sudo apt install tmux
```

-----------------------------------------------------------

## Contributing To The Ceremony
### 1. Create a Temporary Directory
```bash
mkdir ~/p0tion-tmp
cd ~/p0tion-tmp
```

### 2. Switch To Node v16.20
```bash
nvm install 16.20
nvm use 16.20
```

### 3. Create a New Tmux Session
```bash
tmux
```

### 4. Install phase2cli
```bash
npm i @p0tion/phase2cli
```

### 5. Authenticate with GitHub
```bash
npx phase2cli auth
```
This will copy an authentication code to your clipboard, open your web browser, and take you to a GitHub site asking to give p0tion permission to use the “Gists” feature of your GitHub. Paste the contents of your clipboard into this website and click the button to authorize. GitHub will tell you “Congratulations, you’re all set!” and you can return to the terminal.

### 6. Contribute
```bash
npx phase2cli contribute
```

You’ll be asked which ceremony to contribute to.

`RISC Zero STARK-to_SNARK Prover` ceremony should already be highlighted, so press `enter` to select it.

Now, you’ll be asked to choose how to add entropy — this is the secret data for you to destroy and forget (or ideally never know) to secure the ceremony. You can either have it `randomly` generated or create it `manually`. If you don't know what to choose just choose `randomly`.

Once you’ve added your entropy, you’ll be placed in the contribution queue. Contributions take a long time, so leave this running in the background or go do something else. (But don’t let your computer go to sleep, a sleeping computer can’t contribute!)

Now you can detach from the Tmux session. To detach from the current session (leaving it running in the background), press `Ctrl + b`, then release both keys and press `d`.

To reattach to a detached session, type `tmux attach` in your terminal.

> [!NOTE]
> ‍If you run into any errors or lose your connection, you can try to restart your contributions by rerunning the npx phase2cli contribute command. Your contribution should continue where it last left off.

**Once your contribution is complete, you’ll be invited to share a message on Twitter/X — we encourage you to do so, or on whatever social media platform(s) you prefer! 🎉**

### 7. Cleanup
After your ceremony contribution completes, it’s probably a good idea to clean up your files and the GitHub authorization.
```bash
npx phase2cli clean
```
```bash
npx phase2cli logout
```

Go to your `authorized app list on GitHub` and remove the permissions for `pse-p0tion-production`. You can also delete the `~/p0tion-tmp` folder you created if you like.
