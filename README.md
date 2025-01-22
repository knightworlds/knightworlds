<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=1E90FF&height=120&section=header"/>

## 🌟 Welcome to KnightWorlds era!

- ✍ I'm `Solana blockchain developer`. I've developed various tools and `Smart Contracts` to optimize `Trading` strategies, deploy `Tokens`, and manage `Liquidity`.
- 🌱 Built `Rune launchpad`, `Rune Recursive`, `Multi SigWallet`, `Ordinal Marketplace`, `Ordinal Raffle`, `BRC20 Marketplace`, `BTC Defi`, Several `Significant` Projects based on Bitcoin network and `NFT`, `Defi` projects on `EVM` & `Solana Network`

### 📞 Cᴏɴᴛᴀᴄᴛ ᴍᴇ Oɴ ʜᴇʀᴇ:

<p> 
    <a href="https://t.me/knightworlds127" target="_blank"><img alt="Telegram"
        src="https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white"/></a>
</p>

## 🚀 **Recent Projects**

- **`Raydium and Pumpfun Sniper (especially grpc sniper)`**

  Automates tracking of new pools and executes purchases using multiple transaction services.

- **`Raydium and Pumpfun Bundler`**

  Creates a raydium and pumpfun pool and enables dev to buy token in the first block using jito bundling.

- **`Volume Bot in Raydium, Pumpfun and Meteora`**

  Manages market caps and volume of pools with strategic interventions to maintain or increase market cap or liquidity.

- **`Maker Bot (Raydium, Pumpfun)`**

  Increases pool makers by purchasing small tokens from multiple wallets in a short time, working seamlessly with the Volume Bot.

- **`Shit Token Launcher`**

  Deploys new pools on Raydium, leveraging quick sniping bots to generate profits.

- **`Token Freezer`**

  Implements whitelist functionality post-pool creation, restricting token sales to specified users only.

- **`Token Locker Smart Contract`**

  Allows users to stake tokens or SOL with rewards based on staking duration and bonus plans.

- **`Presale-IDO Smart Contract`**

  Facilitates private token markets for crowdfunding, akin to PinkSale's presale mechanism.

- **`BTC Rune Pumpfun`**

  Rune Launchpad, Rune Swap, Buy, Sell, Burn, Multisig Wallet

- **`These are my Solana Smart Contract Code`**
```
use anchor_lang::prelude::*;
use anchor_spl::token::{self, Mint, Token, TokenAccount, Transfer};

declare_id!("program id");

#[program]
pub mod staking {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>, reward_rate: u64) -> Result<()> {
        let staking_account = &mut ctx.accounts.staking_account;
        staking_account.authority = *ctx.accounts.authority.key;
        staking_account.reward_rate = reward_rate;
        staking_account.total_staked = 0;
        Ok(())
    }

    pub fn stake(ctx: Context<Stake>, amount: u64) -> Result<()> {
        let user_account = &mut ctx.accounts.user_account;
        let staking_account = &mut ctx.accounts.staking_account;

        // Transfer tokens from user to staking vault
        token::transfer(
            ctx.accounts
                .transfer_to_vault_context()
                .with_signer(&[]),
            amount,
        )?;

        user_account.staked_amount += amount;
        user_account.last_staked_at = Clock::get()?.unix_timestamp;
        staking_account.total_staked += amount;

        Ok(())
    }

    pub fn claim_rewards(ctx: Context<ClaimRewards>) -> Result<()> {
        let user_account = &mut ctx.accounts.user_account;
        let staking_account = &mut ctx.accounts.staking_account;

        let now = Clock::get()?.unix_timestamp;
        let duration = now - user_account.last_staked_at;
        let rewards = (user_account.staked_amount as u64)
            .checked_mul(staking_account.reward_rate)
            .unwrap_or(0)
            .checked_mul(duration as u64)
            .unwrap_or(0);

        user_account.last_staked_at = now;

        // Mint rewards (assuming rewards come from the staking program mint)
        token::mint_to(
            ctx.accounts
                .mint_to_user_context()
                .with_signer(&[]),
            rewards,
        )?;

        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = authority, space = 8 + 40)]
    pub staking_account: Account<'info, StakingAccount>,
    #[account(mut)]
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Stake<'info> {
    #[account(mut)]
    pub staking_account: Account<'info, StakingAccount>,
    #[account(mut, has_one = staking_account)]
    pub user_account: Account<'info, UserAccount>,
    #[account(mut)]
    pub vault: Account<'info, TokenAccount>,
    #[account(mut)]
    pub user_token_account: Account<'info, TokenAccount>,
    pub token_program: Program<'info, Token>,
}

#[derive(Accounts)]
pub struct ClaimRewards<'info> {
    #[account(mut)]
    pub staking_account: Account<'info, StakingAccount>,
    #[account(mut, has_one = staking_account)]
    pub user_account: Account<'info, UserAccount>,
    #[account(mut)]
    pub reward_mint: Account<'info, Mint>,
    #[account(mut)]
    pub user_token_account: Account<'info, TokenAccount>,
    pub token_program: Program<'info, Token>,
}

#[account]
pub struct StakingAccount {
    pub authority: Pubkey,
    pub reward_rate: u64,
    pub total_staked: u64,
}

#[account]
pub struct UserAccount {
    pub staked_amount: u64,
    pub last_staked_at: i64,
}

impl<'info> Stake<'info> {
    pub fn transfer_to_vault_context(&self) -> CpiContext<'_, '_, '_, 'info, Transfer<'info>> {
        let cpi_accounts = Transfer {
            from: self.user_token_account.to_account_info(),
            to: self.vault.to_account_info(),
            authority: self.user_account.to_account_info(),
        };
        CpiContext::new(self.token_program.to_account_info(), cpi_accounts)
    }
}

impl<'info> ClaimRewards<'info> {
    pub fn mint_to_user_context(&self) -> CpiContext<'_, '_, '_, 'info, MintTo<'info>> {
        let cpi_accounts = MintTo {
            mint: self.reward_mint.to_account_info(),
            to: self.user_token_account.to_account_info(),
            authority: self.staking_account.to_account_info(),
        };
        CpiContext::new(self.token_program.to_account_info(), cpi_accounts)
    }
}

```


