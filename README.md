# Help2Text

## Requrements

- cygwin or myng-w Ruby
- unzip

## Usage

Clone repo

    $git clone https://github.com/leoniv/help_to_text && cd help_to_text

Install gems

    $bundle install

Prepare destination directory for dump

    $mkdir -p tmp/helpdump

Execute

    $bin/help2text 8.3.10 tmp/helpdump

Check result

    $ls -la tmp/helpdump

## Use case

Compare help for different versions.

Prepare destination dir and git

    $mkdir -p tmp/helpdump && git init tmp/helpdump

Dumping first version

    $bin/help2text 8.3.9 tmp/helpdump

Commit dump

    $cd tmp/helpdump
    $git add .
    $git commit -m '8.3.9'

Dump second version

    $../../bin/help2text 8.3.10 ./

Compare

    $git status
    $git diff

or

    $git difftool -t vimdiff


## Enjoy :)
