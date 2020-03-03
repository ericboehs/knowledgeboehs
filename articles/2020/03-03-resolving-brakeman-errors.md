# Resolving Brakeman Errors
After running `brakeman` you'll get a report and, if there are new warnings, it will exit with a status code of "3". Typically, I only see two types of output from Brakeman. Either a large report where there are new warnings or a short report which is saying Brakeman is out of date.

I'll briefly explain resolving both.

## Resolving New Warning(s) Detected
The long report.

Run `brakeman -I` to get an interactive console for ignoring warnings. It will analyze your code base and then prompt you for where to store the ignore file. I've always used the default location of `config/brakeman.ignore`, so I simply press `Enter` to accept the default.

Then I almost always choose option `2` to "Hide previously ignored warnings". This option is a bit confusing. It's the same as option 1 but doesn't show warnings you've already ignored.

At this point, brakeman will iterative over each new warning prompting you to ignore the warning. This is a critical point in your role here. **You need to make sure the warning should be ignored or fixed.** If this is a positive warning, you need to `Ctrl-C` right now and go fix the code rather than ignoring the vulnerability. If it is a false-positive, you can ignore it here via `i` or `n`. I recommend `n` because it prompts for a note as to why this warning is a false-positive.

After you have resolved all warnings, it will (optionally) prompt you to remove fingerprints (outdated warnings). You need to input `y` or `yes` here for it to actually remove the fingerprints.

Finally, you'll want to save changes via `1` and, due to amnesia or disorder, Brakeman will again ask you where the ignore file is. I again just press `Enter` because I use the default location. (I have no idea why the input and output would be in different locations.)

You can run `brakeman` again and it should report "No warnings found". You're all ready to check the ignore file into source control and push it up to your CI.

## Brakeman's Out of Date
The short report.

This only appears if you're running `brakeman --enusre-latest` (which I highly recommend).

To fix, simply run `bundle update brakeman`. This will update your `Gemfile.lock` which is all you should need to change.

If you are locking brakeman in your `Gemfile` (which I don't recommend), you should check Brakeman's [CHANGES][1] to make sure the new changes don't break whatever you're guarding against with the version locking.

[1]:	https://github.com/presidentbeef/brakeman/blob/master/CHANGES.md
