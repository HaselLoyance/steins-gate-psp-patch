# DATA0.AFS
`DATA0.AFS` is a file container. It has an uncompressed archive structure, so this is a good place to explain both AFS and ISO unpacking and repacking.

The general workflow can be described like this:
1. Extract AFS archives from ISO.
2. Unpack files from AFS archive.
3. Modify the files.
4. Repack the AFS archive.
5. Insert the repacked AFS archive into ISO.

#### UMDGen and ISO
By using UMDGen files can be extracted from the game ISO. I'll take `DATA0.AFS` as an example. I will use `I:/` when talking about the files inside of the ISO and usual `/` when talking about files relative to the root of the repo.
**Extracting:**
1. Open the game ISO inside of UMDGen.
2. Navigate to `I:/PSP_GAME/USRDIR`.
3. Right click on the `DATA0.AFS` file in the list and click `Extract Selected`, then choose the destination.

I prefer to keep things organized, so I came up with what I think is a good folder structure.
I have a folder `/DATA0.AFS`, this is where to I'm extracting `DATA0.AFS` from the ISO. But just for convinience sake, I make a copy of it and rename it to `DATA0_ORIGINAL.AFS` (in case I need the original file again).

```
+---DATA0.AFS
|       DATA0.AFS
|       DATA0_ORIGINAL.AFS
|       README.md
```

For obvious reasons (illegal distribution of game content, as well as Github's repository size limits) this folder (and this entire repository) doesn't contain any of the `AFS` files.

**Inserting:**
By using UMDGen files can be inserted into the game ISO. I'll take `DATA0.AFS` as an example.
1. Open the game ISO inside of UMDGen.
2. Go to `File->File List->Export`. Save the text file somewhere (the root of the repository should be fine).
This saves the layout of the ISO into a text file, so that in the future there are no misaligned sectors (where one file might overwrite the other). This also ensures that the files are kept in the proper order (I had some cases where if I messed with the order of the PMF cutscenes they sometimes wouldn't play in the game).
3. Navigate to `I:/PSP_GAME/USRDIR`.
4. Right click on the `DATA0.AFS` file in the list and click `Delete`.
5. Drag and drop a new file (`/DATA0.AFS/DATA0.AFS`).
6. Go to `File->File List->Import`. Open the previously saved file list.
7. *Optional*. Go to `UMD Properties->Optimize`. This will realign the sectors to minimize the ISO size.
7.1. Save the newly rearranged file list. Go to `File->File List->Export`.
8. Go to `Save->Uncomprssed`. This will save the modified ISO file.

Make sure that the name of the new file exactly matches the name of the original file. If this is not the case, then the game most likely is going to break.

#### AFSExplorer and AFS
I'm going to mention once again that AFSExplorer might be not the best choice for you, depending on the game you are working with. As far as I know, it doesn't work with *Chaos;Head Noah*, while [Puyo Tools](https://github.com/nickworonekin/puyotools) works just fine.

By using AFSExplorer the AFS archives can be unpacked. I’ll take `DATA0.AFS` as an example. I will use `A:/` when talking about the files inside of the AFS archive and usual `/` when talking about files relative to the root of the repo.
**Unpacking:**
1. Import AFS archive into AFSExplorer.
1.1. Following from the ISO example above, this archive would be `/DATA0.AFS/DATA0_ORIGINAL.AFS`.
1.2. If you are asked if you want to update the AFS archive structure select `No`.
1.3. Observe that this AFS archive just contains several more AFS archives, which is normal.
2. Right click on the `SCENE00.AFS` file in the list and click `Export`, then choose the destination.

Once again, folder structure is very important.
I have a folder `/SCENE00.AFS`, this is where to I'm exporting the `A:/SCENE00.AFS`. I make a copy of it and rename it to `SCENE00_ORIGINAL.AFS`.

```
+---SCENE00.AFS
|       SCENE00.AFS
|       SCENE00_ORIGINAL.AFS
|       README.md
|       <other files, eg: topographer>
```

Just like you extracted `SCENE00.AFS` from `DATA0.AFS` you can now extract `BIN` files (game scripts) from `SCENE00.AFS`.

**Packing:**
By using AFSExplorer the AFS archives can be repacked. For this example we are going to assume that `/SCENE00.AFS/SCENE00.AFS` was already modified (by using these same instructions of inserting files) and now we need to insert it into `DATA0.AFS`.

1. Import AFS archive into AFSExplorer.
1.1. Following the example from above, this archive would be `/DATA0.AFS/DATA0_ORIGINAL.AFS`.
1.2. If you are asked if you want to update the AFS archive structure select `No`.
2. Right click on the `SCENE00.AFS` file in the list and click `Import`, then choose the new file.
2.1. If you are following the same folder structure, then that file would be `/SCENE00.AFS/SCENE00.AFS`.
2.2. This is where I should warn you about the chunks of AFS archives, as well as how space reservation for files works. Every file has a certain number of free chunks assigned to it. Every chunk is 2048 bytes.
2.2.1. If reserved space for some file is 4096 bytes (2 chunks) but the file takes 2047 bytes (less than 1 chunk), then that means that we have 1 chunk of free space.
2.2.2. If reserved space for some file is 4096 bytes (2 chunks) and the files takes 2049 bytes (more than 1 chunk), then that means that we have no free space.
2.2.3. If `/SCENE00.AFS/SCENE00.AFS` is less in size than `/SCENE00.AFS/SCENE00_ORIGINAL.AFS`, then we are going to have some leftover space in `A:/DATA0.AFS` when importing.
2.2.4. If `/SCENE00.AFS/SCENE00.AFS` is greater in size than `/SCENE00.AFS/SCENE00_ORIGINAL.AFS`, then we are not going to have enough space in `A:/DATA0.AFS` when importing.
2.3. During my work on S;G I've discovered that the game was breaking when there was even 1 chunk of free space, so ideally you want to reserver only as many chunks per file as you need and not more.
**3. If `/SCENE00.AFS/SCENE00.AFS` is greater in size than the original:**
3.1. You'll be prompted with the dialog that there is no room to import the file. When asked if you want to modify reserved space click `Yes`.
3.2. In my tests AFSExplorer was never able to modify the reserved space automatically.
3.3. Go to `Advanced->Modify reserved space`. You should see that the modified reserved space for `SCENE00.AFS` [has changed](https://i.imgur.com/PUJ8RZu.png). Click `Regenerate AFS` and save it with a new name, (eg: `/DATA0.AFS/DATA0_NEW.AFS`).
3.4. Import newly generated archive into AFSExplorer.
3.5. [You should see](https://i.imgur.com/EewJVBW.png) that you now have free space specifically for `SCENE00.AFS`.
3.6. Now you can repeat *step 2*. There should be no warnings. The unused space in the archive should be at 0 bytes.
3.7. The AFS archive doesn't require additional re-saving into a new file. This means that any changes to `/DATA0.AFS/DATA0_NEW.AFS` will be automatically applied.
**4. If `/SCENE00.AFS/SCENE00.AFS` is less in size than the original:**
4.1. Then you don't need to repeat *step 2*, since you have just done that.
4.2. Go to `Advanced->Modify reserved space`. [You should see](https://i.imgur.com/jncOwlo.png) that reserved space for `SCENE00.AFS` is a lot more than the file actually requires.
4.3. Select `SCENE00.AFS` and at the top-right of the windows change how many bytes are reserved for this file. Keep changing it until the `Modified res. slot` is the same `File length`.
4.4. Click `Regenerate AFS` and save it with a new name, eg: `/DATA0.AFS/DATA0_NEW.AFS`).

This concludes AFS and ISO unpacking/repacking.
