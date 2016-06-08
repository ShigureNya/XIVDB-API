# Download Loadstone Icons

Using the XIVDB you can quickly get a list of icon names for lodestone and then mass download them in their 128px glory.

Here is a python script I quickly threw together to do this, it will organize icons into patch folders and they're outputted as "The Name_Icon.png" (done for a wiki format)

```python
import urllib.request as http, os, json, sys

# Fix output format (SE like to keep in microsofts own charset for dashes, quotes, etc...)
def uprint(*objects, sep=' ', end='\n', file=sys.stdout):
    enc = file.encoding
    if enc == 'UTF-8':
        print(*objects, sep=sep, end=end, file=file)
    else:
        f = lambda obj: str(obj).encode(enc, errors='backslashreplace').decode(enc)
        print(*map(f, objects), sep=sep, end=end, file=file)

# XIVDB API Urls
xivdbItems = 'https://api.xivdb.com/item?columns=id,name_en,icon_lodestone,icon_lodestone_hq,patch'
xivdbPatches = 'https://api.xivdb.com/data/patchlist'

# Where do we want to save to
savePath = './icons/'

# Lodestone path
lodestone = 'http://img.finalfantasyxiv.com/lds/pc/global/images/itemicon/'

# Lets count
total = 0
success = 0
failed = 0
skipped = 0

# Get patch list
print('Getting patch list from XIVDB ...')
patchResponse = http.urlopen(xivdbPatches)
patchList = json.loads(patchResponse.read().decode())

# Get item list
print('Getting item list from XIVDB ...')
itemResponse = http.urlopen(xivdbItems)
itemList = json.loads(itemResponse.read().decode())

# Go through items
print('Downloading Icons: %d ' % (len(itemList)))
for item in itemList:
    total += 1

    # Work progress percent (remaining is a bit opposite of what its doing, oh well!)
    remaining = (100 * float(total)/float(len(itemList)))
    remaining = round(remaining, 3)

    # Make sure a lodestone icon exists
    if item['icon_lodestone'] and len(item['icon_lodestone']) > 2:
        # get the patch number for it (dirty but will do)
        patchFolder = 'Unknown/'
        for patch in patchList:
            if patch['patch'] == item['patch']:
                patchFolder = patch['command'] + '/'
                break;

        # Create folder
        newSavePath = savePath + patchFolder;
        if not os.path.exists(newSavePath):
            print('Creating Folder: %s' % (newSavePath))
            os.makedirs(newSavePath)

        try:
            # filenames
            getFilename = '%s%s.png' % (lodestone, item['icon_lodestone'])
            saveFilename = '%s%s_Icon.png' % (newSavePath, item['name_en'])

            # if file doesn't exist download, (if you want to redownload, delete the icon!)
            if os.path.isfile(saveFilename):
                uprint("Skipped:\t%d/%d\t\t%s %%\t\t%s " % (total, len(itemList), remaining, item['name_en']))
                skipped += 1
            else:
                http.urlretrieve(getFilename,saveFilename)
                uprint("Success:\t%d/%d\t\t%s %%\t\t%s " % (total, len(itemList), remaining, item['name_en']))
                success += 1

        except http.HTTPError as err:
            uprint("-Failed:\t%d/%d\t\t%s %%\t\t%s --> %d" % (total, len(itemList), remaining, item['name_en'], err.code))
            failed += 1
    else:
        uprint("Skipped:\t%d/%d\t\t%s %%\t\t%s " % (total, len(itemList), remaining, item['name_en']))
        skipped += 1

print('Icons Downloaded: %d' % (total))
print('Icons Success: %d' % (success))
print('Icons Failed: %d' % (failed))
print('Icons Skipped: %d' % (skipped))

```
