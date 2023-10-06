Fork of [framit-frames](https://github.com/fastlane/frameit-frames) repo to add frames for the iPhone 14 and Samsung Galaxy S21 lineup.

# Setup
1. Use `gem which fastlane` or `sudo gem which fastlane` to locate your fastlane installation (for me it looks like this: `/Users/petertrost/.rvm/gems/ruby-3.3.0-preview2/gems/fastlane-2.216.0/fastlane/lib/fastlane.rb`)
2. Navigate to the path up to and including fastlane-VERSION (for me `cd /Users/petertrost/.rvm/gems/ruby-3.3.0-preview2/gems/fastlane-2.216.0`)
3. From there go to -> frameit -> lib -> frameit and open `frame_downloader.rb` in a text editor
4. Replace the HOST_URL at the top with: `https://peetee06.github.io/frameit-frames/`. It should look like this then:
```
require 'fastlane_core/module'

require_relative 'module'

module Frameit
  class FrameDownloader
    HOST_URL = "https://peetee06.github.io/frameit-frames/"

```
5. Replace the `device_types.rb` file in the same directory with the one from this repo to add the device information for the iPhone 14 lineup.


## Optional
If you want to create your own repo with the new frames mind the following:  

For some reason some of the newer frames now have hyphens between the device name and its color. E.g. `iPhone 14 – Midnight.png`. To fix this you would need to change the `sanitize_filename` method in the Rakefile in `fastlane/frameit/frames_generator` in the fastlane repo to the following before using it to generate the frames:
```ruby
# Facebook's device naming is inconsistent. This method fixes the file names.
def sanitize_filename(filename)
  perform_ipad_renaming(filename
      .gsub('Grey', 'Gray') # some Apple devices have "Grey" instead of "Gray" color -> unify
      .gsub(' - Portrait', '') # iPads Pro include Portrait and Landscape - we just need Portrait; Landscape filtered via DEVICES_TO_SKIP
      .gsub(' - ', ' ') # Google Pixel device names are separated from their colors by a dash -> remove
      .gsub(' — ', ' ') # Some Apple devices have a long dash -> replace
      .gsub(' – ', ' ') # Some Apple devices have a special dash -> replace)
      .gsub('Note10', 'Note 10') # Samsung Galaxy Note 10 is missing a space in "Note10"
      .gsub('Mi Mix Alpha Front', 'Mi Mix Alpha')) # Xiaomi Mi Mix Alpha contains the words "Front", "Back" and "Side" => back and side are filtered via DEVICES_TO_SKIP, "Front" removed from the name here
end
```

You also need to resize the images for the iPhone 14 lineup because they are not pixel accurate. Using them like this would lead to images not being framed correctly. 

To resize them to the correct size I used the screen's pixel width in the below code (e.g. 1170 for iPhone 14) to calculate how I need to resize the images. 

Make sure the screen size in the image is equal to this number. Do not confuse them width the image width because that is larger as it includes the phone frame as well.


# Usage
Make sure you followed the setup steps above before trying to frame your screenshots. 

Follow the official [fastlane frameit](https://docs.fastlane.tools/actions/frameit/) documentation to learn how to use frameit.

# Note

Auto-generated device frames, downloaded and used by [fastlane frameit](https://fastlane.tools).

More information about this on the main repo: https://github.com/fastlane/fastlane/tree/master/frameit/frames_generator
