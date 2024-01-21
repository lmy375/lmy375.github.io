---
layout: links
# multilingual page pair id, this must pair with translations of this page. (This name must be unique)
lng_pair: id_links

# publish date (used for seo)
# if not specified, site.time will be used.
#date: 2022-03-03 12:32:00 +0000

# for override items in _data/lang/[language].yml
#title: My title
#button_name: "My button"
# for override side_and_top_nav_buttons in _data/conf/main.yml
#icon: "fa fa-bath"

# seo
# if not specified, date will be used.
#meta_modify_date: 2022-03-03 12:32:00 +0000
# check the meta_common_description in _data/owner/[language].yml
#meta_description: ""

# optional
# please use the "image_viewer_on" below to enable image viewer for individual pages or posts (_posts/ or [language]/_posts folders).
# image viewer can be enabled or disabled for all posts using the "image_viewer_posts: true" setting in _data/conf/main.yml.
#image_viewer_on: true
# please use the "image_lazy_loader_on" below to enable image lazy loader for individual pages or posts (_posts/ or [language]/_posts folders).
# image lazy loader can be enabled or disabled for all posts using the "image_lazy_loader_posts: true" setting in _data/conf/main.yml.
#image_lazy_loader_on: true
# exclude from on site search
#on_site_search_exclude: true
# exclude from search engines
#search_engine_exclude: true
# to disable this page, simply set published: false or delete this file
#published: false


# you can always move this content to _data/content/ folder
# just create new file at _data/content/links/[language].yml and move content below.
###########################################################
#                Links Page Data
###########################################################
page_data:
  main:
    header: "Blockchain utilities"
    info: "For Blockchain developers and hackers."

  # To change order of the Categories, simply change order. (you don't need to change list order.)
  category:
    - title: "Transaction decoder"
      type: id_txdec
    - title: "Source / Reverse tools"
      type: id_re
    - title: "Data analyzer"
      type: id_data
    - title: "Miscellaneous"
      type: id_misc

  list:
    - type: id_txdec
      title: "Tenderly"
      url: "https://dashboard.tenderly.co/"
      info: "Tenderly is an all-in-one Web3 development platform that accelerates smart contract development and provides a fully integrated developer experience."
    - type: id_txdec
      title: "Phalcon"
      url: "https://phalcon.blocksec.com/explorer"
      info: "Phalcon Explorer is a powerful transaction explorer by BlockSec, supporting Transaction Debugging and Transaction Simulation."
    - type: id_txdec
      title: "Openchain"
      url: "https://openchain.xyz/trace"
      info: "Transaction trace viewer coded by samczsun. Storage difference is supported."
    - type: id_txdec
      title: "Dedaub"
      url: "https://app.dedaub.com/ethereum"
    - type: id_txdec
      title: "Sentio"
      url: "https://app.sentio.xyz/explorer"
      info: "Sentio debugger is a tool to help users understand how a transaction works in detail"
    - type: id_txdec
      title: "Eigenphi"
      url: "https://tx.eigenphi.io/analyseTransaction"
    - type: id_txdec
      title: "EthTx"
      url: "https://ethtx.info/"
      info: "The most classic one. Core code open-sourced."
   
    - type: id_re
      title: "Guess ABI"
      url: "https://abi.w1nt3r.xyz/"
      info: "Get ABI for unverified contracts."
    - type: id_re
      title: "Codeslaw"
      url: "https://www.codeslaw.app/"
      info: "Search for verified smart contracts on ethereum, polygon, arbitrum, optimism, bnbchain, and scroll!"
    - type: id_re
      title: "Bytegraph"
      url: "https://bytegraph.xyz/"
      info: "Generate bytegraph for smart contracts."
    - type: id_re
      title: "yieldfarming diff"
      url: "https://yieldfarming.info/tools/diff/#"
      info: "Ethereum Contract DiffChecker."
    - type: id_re
      title: "Openchain"
      url: "https://openchain.xyz/signatures"
      info: "Openchain Signature Database."
    - type: id_re
      title: "4byte.directory"
      url: "https://www.4byte.directory/"
      info: "Ethereum Signature Database."

    - type: id_data
      title: "Dune"
      url: "https://dune.com/"
    - type: id_data
      title: "Flipside"
      url: "https://flipsidecrypto.xyz/"
    - type: id_data
      title: "Chainbase"
      url: "https://console.chainbase.com/"

    - type: id_misc
      title: "Upgradehub"
      url: "https://upgradehub.xyz/"
      info: "Proxy upgrade information"
    - type: id_misc
      title: "Louper"
      url: "https://louper.dev/"
      info: "Louper - The Ethereum Diamond Inspector"
    - type: id_misc
      title: "Metasleuth"
      url: "https://metasleuth.io/"
      info: "Crypto Tracking and Investigation Platform by BlockSec"
    - type: id_misc
      title: "PrivateKeyFinder"
      url: "https://privatekeyfinder.io/"
      info: "Private keys database"
    - type: id_misc
      title: "Mempool explorer"
      url: "https://explorer.blocknative.com/"
      info: "Blocknative mempool explorer"
    - type: id_misc
      title: "DexScreener"
      url: "https://dexscreener.com/"
      info: "Realtime DEX analytics & charts"
      

      
     
---
