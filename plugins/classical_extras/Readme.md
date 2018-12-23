# General Information
This is the documentation for version 2.0.2 of "classical\_extras". There may be beta versions later than this - check [my github site](https://github.com/MetaTunes/picard-plugins/tree/metabrainz/2.0/plugins/classical_extras) for newer releases. For further help, please review [the forum thread](https://community.metabrainz.org/t/classical-extras-2-0/394627) or post any new questions there. It only works with Picard version 2.0, **NOT** earlier versions. If you are using Picard 1.4.x, please choose the ["1.0" branch on github](https://github.com/MetaTunes/picard-plugins/tree/1.0/plugins/classical_extras) and use the latest release there - also use the [earlier forum thread](https://community.metabrainz.org/t/classical-extras-plugin/300217).

This version has only been tested with FLAC and mp3 files. It does work with m4a files, but Picard does not write all m4a tags (see further notes for iTunes users at the end of the "works and parts tab" section). "Classical Extras" populates tags and hidden variables in Picard with information from the MusicBrainz database about the recording, artists and work(s), and of any containing works, passing up through multiple work-part levels until the top is reached. The "Options" page (Options->Options->Plugins->Classical Extras) allows the user to determine how hidden variables are written to file tags, as well as a variety of other options.

This plugin is particularly designed to assist with tagging of classical music so that player or library manager software, which can display multiple work levels, different artist types and custom tags, can have access to these details.  
It has two main components - "Artists" and "Works and parts" - which can be used independently or together. "Works and parts" will take at least as many seconds to process as there are works to look up (owing to MusicBrainz  throttling) so users who only want the extra artist information and the tag-mapping feature, but not the work details, may turn it off (e.g. perhaps for 'popular' music).  There are also two tabs - "Genres etc." and "Tag mapping" which may be used provided either "Artists" or "Works and parts" (or both) are run. Finally, an "Advanced" tab contains additional options.

Hidden metadata variables produced by this plugin are (mostly) prefixed with "\_cwp\_" or  "\_cea\_" depending on which component of the plugin created them. Full details of these variables are given in a later section.
Tags are output depending on the choices specified by the user in the Options Page. Defaults are provided for these tags which can be added to / modified / deleted according to user requirements. 
If the Options Page does not provide sufficient flexibility, users familiar with scripting can write Tagger Scripts to access the hidden variables directly.

## Updates
Version 2.0.2: Changed layout of tabs - the order is now Artists, Works, Genres and Tag-mapping. The help tab has been much reduced as it was of limited assistance and difficult to maintain. There is a lot of context-sensitive help and  the readme file contains the latest full documentation.  
Added a check-box on the genres tab to enable/disable genre filtering (previously the genre names would have to be blank to eliminate filtering and in any case this was buggy).   
A general code tidy-up has led to significant performance enhancements. The only significant slowing factor is now the unavoidable MusicBrainz 1 look-up per second constraint (for looking up works).   
A number of changes have been made to the way in which "extended" metadata is supplied - i.e. where title metadata is combined with the "canonical" MusicBrainz work names. Hopefully the result is a more consistent and helpful presentation. Also some minor changes to the way in which text is eliminated to arrive at part names, including a new option on the "advanced" tab to "Allow blank part names for arrangements and part recordings, if an arrangement/partial label is provided" (see documentation under the advanced tab section for more details). Plus bug fixes.

Version 2.0.1: Minor update to add _composer_lastnames variable.

Version 2.0: Major overhaul of version 0.9.4 to achieve Picard 2.0 and Python 3.7 compatibility. All webservice calls re-written for JSON rather than XmlNode responses. Periods are written to tag in date order. Addition of sub-options for inclusion of key signature information in work names. If the MB database has circular work references (i.e a parent is a descendant of itself) then these will be trapped, ignored and reported. Numerous small refinements, especially of text comparison algorithms (e.g. option to control removal of prepositions - see advanced tab). Bug fixes.

For a list of previous version changes, see the end of this document.

# Installation
Install the zip file in your plugins folder in the usual fashion.

# Usage
After installation, go to the Options Page and modify choices as required. There are 5 tabs - "Artists", "Works and parts", "Genres etc.", "Tag mapping" and "Advanced". The sections below describe each of these. If the options provided do not allow sufficient flexibility for a user's need (hopefully unlikely!) then Tagger Scripts may be used to process the hidden variables or other tags. Alternatively, it may be possible to achieve the required result by running and saving twice (or more!) with different options each time. This is not recommended for more than a one-off - a script would be better.

**Important**: 
1.  The plugin **will not work fully unless** "Use release relationships" and "Use track relationships" are enabled in Picard->Options->Metadata. The plugin will enable these options by default when starting Picard. However, it may be that the MusicBrainz database has conflicting data between track and release relationships, in which case you may wish to temporarily turn off one of these options, but it is better to fix the incorrect data using "Edit relationships" in MusicBrainz.  
2.  It is recommended only to use the plugin on one or a few release(s) at a time, particularly for initial tagging if the "Works and parts" function is being used. The plugin is not designed to do "bulk tagging" of untagged files - it may be better to use a tool such as SongKong for that and then use the plugin to enhance the results as required. However, once you have tagged files (either in Picard or another tool) such that they all have MusicBrainz IDs, you should be able to re-tag multiple releases by dragging the containing folder into Picard; this is useful to pick up changed MusicBrainz data or if you change the Classical Extras version or options (but bear in mind that the "Works and parts" function will still take at least 1 second per track.  
3.  **Check for error messages before saving a release**. The plugin will write out special "error message" tags which should appear prominently in the bottom Picard pane. In particular, look for "000\_major\_warning" and "001\_errors". please read the messages carefully and follow any recommended actions.

    **Watch out for "002\_important\_warning" - "No file with matching trackid - IF THERE SHOULD BE ONE, TRY 'REFRESH' - (unable to process any saved options, lyrics or 'keep' tags)"; this will always occur if you load a file without MusicBrainz ids - just refresh it to pick up any existing file tags such as lyrics, if required.**  This will also occur if you have manually matched files rather than used Picard's "lookup" or "scan" functions. It may also be due to Picard processing issues - more likely if the files are on a network server; if you are getting it a lot then it may be better to move the files onto local storage to do the updates. 
4.  If you are just changing option settings then you can usually "use cache" (see "work and parts" tab section 1) when refreshing, to avoid the 1-second per work delay. However, if the works data in MusicBrainz has been changed then obviously you will need to do a full look-up, so disable cache. If the work structure has been fundamentally changed (i.e. a different hierarchy of existing works) - either within the MusicBrainz database or by selecting/deselecting the "include collection relations", partial" or "arrangements" options - then you may need to quit and restart Picard to correctly pick up the new structure.  
5.  Keep a backup of your picard.ini file (C:\Users\[user name]\AppData\Roaming\MusicBrainz in Windows) in case you erase your settings or Picard crashes and loses them for you.

## Artists tab
There are five coloured sections as shown in the screen image below:

![Artist options](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/artists/)

1. "Create extra artist metadata" should be selected otherwise this section will not run. This is the default.

2. "Work-artist/performer naming options". 
  This section deals primarily with the application of aliases and "credited as" names to replace the MusicBrainz standard names. The first box allows you to choose whether to replace MusicBrainz standard  names by aliases - either for all work-artists/performers or only work-artists (writers, composers, arrangers, lyricists etc.). The second box sets the usage of "credited as" names: the first part of this lists all the places where "credited as" names can occur (really!) and the second part allows you to apply these to performing artists and/or work-artists. 

   Please note that, in the current version of this plugin, only aliases and "credited as" names which are in the "release XML node" are available (i.e. roughly those relating to the metadata shown in the release overview page in MusicBrainz). So, for example, if a recording is an arrangement of another work and that other work (but not the arrangement itself) has a composer linked to it, then the composer's alias will not be available (nor is the composer shown on the MB release overview page). In some cases (if appropriate) this can be remedied by adding the relevant composer relationship to the lowest-level work.

    >Note regarding aliases and "credited as" names:  
    In a MB release, an artist can appear in one of seven contexts. Each of these is accessible in releaseXmlNode
    and the track and recording contexts are also accessible in trackXmlNode.  
    (They are applied in sequence - e.g. track artist credit will over-ride release artist credit)  
    The seven contexts are:  
    Recording: credited-as and alias (this is applied first as it is the most general - i.e. it may apply to more than one release)  
    Release-group: credited-as and alias  
    Release: credited-as and alias  
    Release relationship: credited-as only (see note)  
    Recording relationship (direct): credited-as only (see note)  
    Recording relationship (via work): credited-as only (see note)  
    Track: credited-as and alias  
    Note: Aliases **may** be retrieved for "relationship" artists, but the retrieval is not reliable (MusicBrainz webservice issue)

    N.B. if more than one release is loaded in Picard, any available alias names loaded so far will be available and used. However, "credited as" names will only be used from the current release. If you do not want these names to be available then you may need to restart Picard after changing the option settings (otherwise they will still be cached).

    In addition to the plugin options, the main Picard options also have an effect on how 'track artists' (or any tags derived from them through tag-mapping) are displayed. In Options->Metadata, if "Translate artist names..." is selected then the alias will be used for the track artist (or failing that, a name based on the sort-name), rather than the "credited as" name. If "Use standardized artist names" is selected then neither the alias nor the "credited as" name will be used. In order to facilitate consistency, Classical Extras will save these Picard options along with its own options in specific tags (see "Advanced options" section 6).

     The bottom box then (a) allows a choice as to whether aliases will over-ride "credited as" names or vice versa and (b) whether, if there are still some names in non-Latin script, these should be replaced (this will always remove middle [patronymic] names from Cyrillic-script names [but does not deal fully with other non-Latin scripts]; it is based on the sort names wherever possible).

     Note that **none of this processing affects the contents of the "artist" or "album\_artist" tags, unless they are replaced by a "tag mapping" action**. These tags may be either work-artists or performing artists. Their contents are determined by the standard Picard options "translate artist names" and "use standardized artist names" in Options-->Metadata. If "translate name" is selected, the name will be the alias or (if no alias) the 'unsorted' sort-name; otherwise the name will be the MusicBrainz name if "use standardized artist names" is selected or the "credited as" name (if available) if it is not selected. Using the saved options to over-ride the displayed options has no effect on this as the processing takes place in Picard itself, not the plugin (but **over-write** will over-write all Picard and plugin options with the saved ones).

3. "Recording artist options".
  In MusicBrainz, the recording artist may be different from the track artist. For classical music, the MusicBrainz guidelines state that the track artist should be the composer; however the recording artist(s) is/are usually the principal performer(s).  
  Because, in classical music (in MusicBrainz), recording artists will usually be performers whereas track artists are composers, by default the naming convention for performers (set in the previous section) will be used (although only the as-credited name set for the recording artist will be applied). Alternatively, the naming convention for track artists can be used - which is determined by the main Picard metadata options.

    Classical Extras puts the recording artists into 'hidden variables' (as a minimum) using the chosen naming convention.
  There is also option to allow you to replace the track artist by the recording artist (or to merge them). The chosen action will be applied to the 'artist', 'artists', 'artistsort' and 'artists\_sort' tags. Note that 'artist' is usually a single-valued string with a "join phrase" such as a semi-colon for multiple artists, whereas 'artists' is a list and may be multi-valued. Lists are simply merged but, because the 'artist' string may have different join-phrases etc, a merged tag may have the recording artist(s) in brackets after the track artist(s). Obviously, for classical music, if you use "merge" then the artist tag will have both the composer and the recording artists: this may be desirable for simple players (with no composer recognition) but otherwise may look odd.

     Note that, if the original track artist is required in tag mapping (i.e. as it was before replacement/merge with recording artist), it is available through the hidden variable \_cea\_MB\_artists.

     Note also that, if @loujin's browser script has been used to fill the recording artist data, this will be the same as the performing artists in the Recording-Artist relationship - i.e. it may be a lengthy list rather than the principal artist for the track.

4. "Other artist options":

    "Modify host tags and include annotations" (Previously called "Include arrangers from all work levels"). This will gather together, for example, any arranger-type information from the recording, work or parent works and place it in the "arranger" tag ('host' tag), with the annotation (see below) in brackets. All arranger types will also be put in a hidden variable, e.g. \_cwp\_orchestrators. The table below shows the artist types, host tag and hidden variable for each artist type.
        <table>
        <tr><td>Artist type</td><td>Host tag</td><td>Hidden variable</td></tr>
        <tr><td>writer</td><td>composer</td><td>writers</td></tr>
        <tr><td>lyricist</td><td>lyricist</td><td>lyricists</td></tr>
        <tr><td>revised by</td><td>arranger</td><td>revisors</td></tr>
        <tr><td>translator</td><td>lyricist</td><td>translators</td></tr>
        <tr><td>arranger</td><td>arranger</td><td>arrangers</td></tr>
        <tr><td>reconstructed by</td><td>arranger</td><td>reconstructors</td></tr>
        <tr><td>orchestrator</td><td>arranger</td><td>orchestrators</td></tr>
        <tr><td>instrument arranger</td><td>arranger</td><td>arrangers (with instrument type in brackets)</td></tr>
        <tr><td>vocal arranger</td><td>arranger</td><td>arrangers (with voice type in brackets)</td></tr>
        <tr><td>chorus master</td><td>conductor</td><td>chorusmasters</td></tr>
        <tr><td>concertmaster</td><td>performer (with annotation as a sub-key)</td><td>leaders</td></tr>
        </table>

    If you want to be more selective as to what is included in host tags, then disable this option and use the tag mapping section to get the data from the hidden variables. If you want to add arrangers as composers, do so in the tag mapping section also. 

     (Note that Picard does not normally pick up all arrangers, but that the plugin will do so, provided the "Works and parts" section is run.)

     "Name album as 'Composer Last Name(s): Album Name'" will add the composer(s) last name(s) before the album name, if they are listed as album artists. If there is more than one composer, they will be listed in the descending order of the length of their music on the release. MusicBrainz style is to exclude the composer name unless it is actually part of the album name, but it can be useful to add it for library organisation. The default is checked.
     
     "Do not write 'lyricist' tag if no vocal performers". Hopefully self-evident. This applies to both the Picard 'lyricist' tag and the related internal plugin hidden variables '\_cwp\_lyricists' etc. 

     Note that the plugin will search for lyricists at all work levels (bottom up), but will stop after finding the first one (unless that was just a translator).

     "Do not include attributes in an instrument type" (previously just referred to the attribute 'solo'). MusicBrainz permits the use of "solo", "guest" and "additional" as instrument attributes although, for classical music, its use should be fairly rare - usually only if explicitly stated as a "solo" on the the sleevenotes. Classical Extras provides the option to exclude these attributes (the default), but you may wish to enable them for certain releases or non-Classical / cross-over releases.

     "Annotations": The chosen text will be used to annotate the artist type within the host tag (see table above for host tags), but only if "Modify host tags" is selected.

     Please note that the use of the word "master" is the MusicBrainz term and is not intended to be gender-specific. Users can specify whatever text they please.

5. "Lyrics". **Please note that this section operates on the underlying input file tags, not the Picard-generated tags (MusicBrainz does not have lyrics)**
  Sometimes "lyrics" tags can contain album notes (repeated for every track in an album) as well as track notes and lyrics. This section will filter out the common text for a release and place it in a different tag from the text which is unique to each track.

   "Split lyrics tag": enables this section.

   "Incoming lyrics tag": The name of the lyrics file tag in the input file (normally just 'lyrics').

   "Tag for album notes": The name of the tag where common text should be placed.

   "Tag for track notes": The name of the tag where notes/lyrics unique to a track should be placed.

   Note that if the 'output' tags are not specified, then internal 'hidden' variables will still be available for use in the tag-mapping section (called album\_notes and track\_notes).

## Work and parts tab

There six coloured sections as shown in the screen print below:

![Works and parts options](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/work-parts/)

1. "Include all work levels" should be selected otherwise this section will not run. This is the default.

    "Include collection relationships" (selected by default) will include parent works where the relationship has the attribute 'part of collection'. See [Discussion](https://community.metabrainz.org/t/levels-in-the-structure-of-works/293047/109) for the background to this. Note that only "work" entity types will be included, not "series" entities. If this option is changed, it will not take effect on releases already loaded in Picard - you will need to quit and restart. PLEASE BE CONSISTENT and do not use different options on albums with the same works, otherwise you may not get what you want.

    "Use cache (if available)" prevents excessive look-ups of the MB database. Every look-up of a work needs to be performed separately (hopefully the MB database might make this easier some day). Network usage constraints by MB means that each look-up takes a minimum of 1 second. Once a release has been looked-up, the works are retained in cache, significantly reducing the time required if, say, the options are changed and the data refreshed. However, if the user edits the works in the MB database then the cache will need to be turned off temporarily for the refresh to find the new/changed works. Also some types of work (e.g. arrangements) will require a full look-up if options have been changed. **Do not leave this option turned off** as it will make the plugin slower and may cause problems. This option will always be set on when Picard is started, regardless of how it was left when it was last closed.

2. "Tagging style". This section determines how the hierarchy of works will be sourced.

    * **Works source**: There are 3 options for determing the principal source of the works metadata
      - "Use only metadata from title text". The plugin will attempt to extract the hierarchy of works from the track title by looking for repetitions and patterns, using the work structure in MusicBrainz as a guide. If the title does not contain all the work names in the hierarchy then obviously this will limit what can be provided.
      - "Use only metadata from canonical works". The names from the hierarchy in the MB database will be used. Assuming the work is correctly entered in MB, this should provide all the data. However the text may differ from the track titles and will be the same for all recordings. It may also be in the language of the composer whereas the titles will probably be in the language of the release. (This language issue can also be addressed by using aliases - see below).
      - "Use canonical work metadata enhanced with title text". This supplements the canonical data with text from the titles **where it is significantly different**. The supplementary title data will be in curly brackets. This is clearly the most complete metadata style of the three but may lead to long descriptions. It is particularly useful for providing translations - see image below for an example (using the Muso library manager). In this example, title text that is similar to that in the canonical text has been eliminated to make the text shorter - the mannr of doing this is controlled by settings on the Advanced tab.

      ![Respighi](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/respighi/)

    * **Source of canonical work text**. Where either of the second two options above are chosen, there is a further choice to be made:
      - "Full MusicBrainz work hierarchy". The names of each level of work are used to populate the relevant tags. E.g. if ""Concert Fantasy for Piano and Orchestra, op. 56: I. Quasi Rondo" (level 0) is part of "Concert Fantasia, op. 56" (level 1) then that is how they will appear, since there is no repetition of text between parent and child. So, while accurate, this option might sometimes be rather verbose.
      - "Consistent with lowest level work description". The names of the level 0 work are used to populate the relevant tags. So, in the above example, "Concert Fantasy for Piano and Orchestra, op. 56" will be shown as the work and "I. Quasi Rondo" will be shown as the movement. Sometimes this may look better, but not always, **particularly if the level 0 work name does not contain all the parent work detail**. If the full structure is not implicit in the level 0 name then a warning will be logged and written to the "warning" tag.
      
      **Version 2.0 update**: the second option is needed less often now as there is a more sophisticated matching algorithm for the canonical work names. Text may be eliminated at places other than the start, and synonyms may be used to achieve greater matching. These options are set on the Advanced tab. Setting "Removal of common text between parent and child works" to 2 (the default) and including "Fantasia" as a synonym of "Fantasy" yields the following result:  
      Work: "Concert Fantasia, op. 56", Movement: "for Piano and Orchestra, … : I. Quasi Rondo"  
      This still repeats "for Piano and Orchestra" for each movement as this text is in level 0, not level 1 (where it only appears as disambiguation). Arguably the best way to fix this is to have consistent work names in MB. (Of course, this specific example may have been fixed in MB by now, but the principle still holds). The strategy below has been updated to reflect this

   **Strategy for setting style:** *It is suggested that you start with "extended/enhanced" style and the "Full MusicBrainz work hierarchy" as the source (this is the default) and tweak the advanced settings if necessary. If this does not give acceptable results, try switching to "Consistent with lowest level work description". If the "enhanced" details in curly brackets (from the track title) give odd results then, again, try tweaking the advanced settings (see later section) or switch the style to "canonical works" only. Any remaining oddities are probably in the MusicBrainz data, which may require editing.*

3. "Aliases"

     "Replace work names by aliases" will use **primary** aliases for the chosen locale instead of standard MusicBrainz work names. To choose the locale, use the drop-down under "translate artist names" in the main Picard Options-->Metadata page. Note that this option is not saved as a file tag since, if different choices are made for different releases, different work names may be stored and therefore cannot be grouped together in your player/library manager. The sub-options then allow either the replacement of all work names, where a primary alias exists, just the replacement of work names which are in non-Latin script, or only replace those which are flagged with user "Folksonomy" tags. The tag text needs to be included in the text box, in which case flagged works will be 'aliased' as well as non-Latin script works, if the second sub-option is chosen. Note that the tags may either be anyone's tags ("Look in all tags") or the user's own tags. If selecting "Look in user's own tags only" you **must** be logged in to your MusicBrainz user account (in the Picard Options->General page), otherwise repeated dialogue boxes may be generated and you may need to force restart Picard.

4. "Tags to create" sets the names of the tags that will be created from the sources described above. All these tags will be blanked before filling as specified. Tags specified against more than one source will have later sources appended in the sequence specified, separated by separators as specified.

    * **Work tags**:
      - "Tags for Work - for software with 2-level capability". Some software (notably Muso) can display a 2-level work hierarchy as well as the work-movement hierarchy. This tag can be use to store the 2-level work name (a double colon :: is used to separate the levels within the tag).
      - "Tags for Work - for software with 1-level capability". Software which can display a movement and work (but no higher levels) could use any tags specified here. Note that if there are multiple work levels, the intermediate levels will not be tagged. Users wanting all the information should use the tags from the previous option (but it may cause some breaks in the display if levels change) - alternatively the missing work levels can be included in a movement tag (see below).
      - "Tags for top-level (canonical) work". This is the top-level work held in MB. This can be useful for cataloguing and searching (if the library software is capable).

    * **Movement/Part tags**:
      (a) "Tags for(computed) movement number". This is not necessarily the embedded movt/part number, but is the sequence number of the movement within its parent work **on the current release**.  
      (b) "Tags for Movement - excluding embedded movt/part numbers". As below, but without the movement part/number prefix (if applicable)  
      (c) "Tags for Movement - including embedded movt/part numbers". This tag(s) will contain the full lowest-level part name extracted from the lowest-level work name, according to the chosen tagging style.  
      For options (b) and (c), the tags can either be filled "for use with multi-level work tags" or "for use with 1-level work tags (intermediate works will prefix movement)" - or different tags for each column.  The latter option will include any intermediate work levels which are missing from a single-level work tag. Use different tag names for these, from the multi-level version, otherwise both versions will be appended, creating a multi-valued tag (a warning will be given).  
      Note that if a tag is included in (a) and either of (b) or (c), the movement number will be prepended at the beginning of the tag, followed by the selected separator.

   **Strategy for setting tags:** *It is suggested that initially you just use the multi-level work tag and related movement tags, even if your software only has a single-level work capability. This may result in work names being repeated in work headings, but may look better than the alternative of having work names repeated in movement names. This is the default.* 

   *If this does not look good you can then compare it with the alternative approach and change as required for specific releases. If your software does not have any "work" capability, then you can still get the full work details by, for example, specifying "title" as both a work and a movement tag.*

5. "Partial recordings, arrangements and medleys" gives various options where recordings are not just simply of a named complete work. These only apply if one of the two "canonical work" styles is in operation (i.e. not if "Use only metadata from title text" is selected).

    * **Partial recordings**:
      If this option is selected, partial recordings will be treated as a sub-part of the whole recording and will have the related text (in the adjacent box) included in its name. Note that this text is placed at the start of the canonical name, but the latter will probably be stripped from the sub-part as it duplicates the recording work name; any title text (for "extended" style) will be appended to the whole. Note that, if "Consistent with lowest level work description" is chosen in section 2, the text may be treated as a "prefix" similar to those in the "Advanced" tab. If this eliminates other similar prefixes and has unwanted effects, then either change the desired text slightly (e.g. surround with brackets) or use the "Full MusicBrainz work hierarchy" option in section 2.  Note that similar text between the partial work and its 'parent' will be removed which will frequently result in no text other than the specified 'partial text', unless extended metadata is used resulting in appended text in {} - this behaviour can be controlled by disabling the setting "Allow blank part names for arrangements and part recordings..." on the advanced tab.

    * **Arrangements**:
      If this option is selected, works which are arrangements of other works will have the latter treated in the same manner as "parent" works, except that the arrangement work name will be prefixed by the text provided. Note that similar text between the arranged work and its parent will be removed unless this results in no text, in which case a stricter comparison (as for the derivation of 'part' names from works) will be used.
      
    **Important note:** *If the Partial or Arrangement options are changed (i.e. selected/deselected) then quit and restart Picard as the work structure is fundamentally different. If the related text (only) is changed then the release can simply be refreshed.*

    * **Medleys**
      These can occur in two ways in MusicBrainz: (a) the recording is described as a "medley of" a number of works and (b) the track is described as (more than one) "medley including a recording of" a work. See [Homecoming](https://musicbrainz.org/release/393913a2-7fde-4ed5-8be6-ca5c2c0ccf0d) for examples of both (tracks 8, 9 and 11). In the first case, the specified text will be included in brackets after the work name, whereas in the second case, the track will be treated as a recording of multiple works and the specified text will appear in the **parent** work name.

6. "SongKong-compatible tag usage".

    "Use work tags on file (no look up on MB) if Use Cache selected": This will enable the existing work tags on the file to be used in preference to looking up on MusicBrainz, if those tags are SongKong-compatible (which should be the case if SongKong has been used or if the SongKong tags have been previously written by this plugin). If present, this can speed up processing considerably, but obviously any new data on MusicBrainz will be missed. For the option to operate, "Use cache" also needs to be selected. Although faster, many of the subtleties of a full look-up will be missed - for example, parent works which are arrangements will not be highlighted as such, some arrangers or composers of original works may be omitted and some medley information may be missed. Other information, such as composed-dates will also be missing. **In general, therefore, the use of this option will result in poorer metadata than allowing the full database look-up to run. It is not recommended unless you have already tagged your files with SongKong and speed is more important than quality.**

    "Write SongKong-compatible work tags" does what it says. These can then be used by the previous option, if the release is subsequently reloaded into Picard, to speed things up (assuming the reload was not to pick up new work data). The same caveats as those above apply.
    
    **Note that, as from version 2.0.2, there is no significant speed difference between (basic) SongKong and Picard with Classical Extras, so this feature is only ever useful if the album has already been tagged with SongKong.**

    The default for both these options is unchecked.

    Note that Picard and SongKong use the tag musicbrainz\_workid to mean different things. If Picard has overwritten the SongKong tag (not a problem if this plugin is used) then a warning will be given and the works will be looked up on MusicBrainz. Also note that once a release is loaded, subsequent refreshes will use the cache (if option is ticked) in preference to the file tags.

**Note for iTunes users:** *iTunes and Picard do not work well together. iTunes can display work and movement for m4a(mp4) files, but Picard does not write the movement tag. To work round this, write the movement to the "subtitle" tag assuming that is not otherwise used, and use a simple Mp3tag action to convert it to MOVEMENTNAME before importing to iTunes. If you are writing to a FLAC file which will subsequently be converted to m4a then different tag names may be required; e.g. using dBpoweramp, write the movement to "movement name". In both cases use "work" for the work. To store the top\_work, use "grouping" if writing directly to m4a, but "style" if writing to FLAC followed by dBpoweramp conversion. You can put multiple tags into the boxes described above so that your options are multi-purpose. N.B. if work tags are specified and the work has at least one level (i.e. at least work: movement), then the tag "show work movement" will be set to 1. This is used by iTunes to trigger the hierarchical display and should work both directly with m4a files and indirectly via files which are subsequently converted.*

## Genres etc. tab

This section is dependent on both the artists and workparts sections. If either of those sections are not run then this section will not operate correctly. At the very top of the tab is a checkbox "Use Muso reference database...". For [Muso](http://klarita.net/muso.html) users, selecting this enables you to use reference data for genres, composers and periods which have been entered in Muso's "Options->Classical Music" section. Regardless as to whether this is selected, there are then three main coloured sections, each with a number of subsections. The details in each section differ depending on whether the "Muso" option is selected.  The screen print below shows the options assuming it is not selected (differences occurring when "Muso" is selected are discussed later):

![Genres etc.](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/genres-plain/)

1. "Genres". Two separate tags may be used to store genre information, a main genre tage (usually just "genre") and a sub-genre tag. These need to be specified at the top of the section. If either is left blank then the related processing will not run.

    * **Source of genres**
    Any or all of four sources may be selected. In each case, any values found are treated as "candidate genres" - they will only be applied to the specified genre and sub-genre tags in accordance with the criteria in the "allowed genres" section, if any (see below).
      
      (a) "Existing file tag". The contents of the existing file tag (as specified above - main genre tag only) will be included as candidate genres. Note that, if this tag name is not "genre", then the contents of the tag "genre" will be included as well.
      
      (b) "Folksonomy work tags". This will use the folksonomy tags for **works** (including parent works) as a possible source of genres. To use the folksonomy tags for **releases/tracks**, select the main Picard option in Options->Metadata->"Use folksonomy tags as genre". Again (unlike vanilla Picard) these are candidate genres, and will only be published if they match the allowed genres.
      
      (c) "Work-type". The work-type attribute of works or parent works will be used as a candidate genre.
      
      (d) "Infer from artist metadata". This option was on the artist tab in version 0.9.1 and prior. Owing to the additional genre processing now available, the operation of this option is slightly restricted compared to the earlier versions. It attempts to create candidate genres based on information in the artist-related tags. Values provided are:
	Orchestral, Concerto, Choral, Opera, Duet,Trio, Quartet, Chamber music, Aria ('classical values') and Vocal, Song, Instrumental ('generic values'). If the track is a recorded work and the track artist is the composer (i.e. MusicBrainz 'classical style'), the candidate genre values will also include "Classical". The 'classical values' will only be included as candidate genres if the track is deemed to be 'classical' by some part of the genre processing section.
	
	* **Allowed genres**
	A check-box (ticked by default) enables this section. If it is unchecked, then no genre filtering is applied - all 'candidate genres' will be written to the genre tab.  
	Four boxes are provided for lists of genres which are "allowed" to appear in the specified tags. Each list should be comma-separated (and no commas in any genre name). Candidate genres matching those in a "main genre" box will be added to the specified main genre tag. Similarly for sub-genres. If a candidate genre matches a 'classical genre' (in one of the top two boxes), then the track will be deemed to be "Classical" (see next part for more details).  
You may also enter a genre name to be used if no matching main genre is found (otherwise the tag will be blank). 

    * **"Classical" genre**
     Normally (i.e. by default) a work will only be deemed to be 'classical' if it is inferred from the MusicBrainz style (see "source of genres") or if a candidate genre matches a "Classical" genre or sub-genre list. However, you may select that all tracks are 'classical' regardless. There is also an option to exclude the word "Classical" from any genre tag, but still treat the work as classical. If a work is deemed to be classical, a tag may be written with a specified value as set out in the last two boxes of this section. For example, to be consistent with SonKong/Jaikoz, you could set "is\_classical" to "1".  
  
2. "Instruments and keys".
    * **Instruments**
    Specify the tag name you wish instrument names to appear in. Instruments will be sourced from performer relationships. Instrument names may either be the standard MusicBrainz names or the "credited as" names in the performer relationship, or both. Vocal types are treated similarly to instruments. (Note that, in v0.9.1 and prior, instruments were written to the same tag as inferred genres. If you wish to continue this, then you may use the same tag name here as for the genre tag.)
    
    * **Keys**
    Specify the tag name in which you wish the key signatures of works to appear. Keys will be obtained from all work levels (assuming these have been looked up): for example, Dvořák's Largo From the New World will be shown as D♭ major, C# minor (the main keys of the movement) and E minor (the home key of the overall work).  
"Include key(s) in work names" gives the option to include the key signature for a work in brackets after the name of the work in the metadata. Keys will be added in the appropriate levels: e.g. Dvořák's New World Symphony will get (E minor) at the work level, but only movements with different keys will be annotated viz. "II. Largo (D-flat major, C-Sharp minor)". The default sub-option is to only add these details if the key signature is missing from the work title (other sub-options are to never or to always include the information.  
    
3. "Periods and dates".

    * **Work dates**
    Specify the tag name to hold work dates. Work dates will be given as a "year" value only, e.g. "1808" or a range: "1808-1810". The sources of these dates is specified in the next part. If the movement has a composed date(s), this will be used, otherwise the the dates from the parent work will be used (if available).
    
        "Source of work dates". Select which sources to use - from composed, published and premiered, then decide whether to use them in preferential order (e.g. if "composed date" exists, then the others will not be used) or to show them all.
    
        "Include workdate in work name ..." operates analogously to  "Include key(s) in work names" described above. (Work dates will be used in preference order, i.e. composed - published - premiered, with only the first available date being shown).
    
    * **Periods**
    This section will use work dates, where available, to determine the "classical period" to which it belongs, by means of a "period map" (Muso users can also use composer dates - see below). 
    
        Specify the tag name to hold the period data. The period map should then be entered in the format "Period name, Start\_year, End\_year; Period name2, Start\_year, End\_year;"  etc. Periods may overlap. Do not use commas or semi-colons within period names. Start and end years must be integers.
    
## Genres etc. tab - Muso-specific processing

Users of [Muso](http://klarita.net/muso.html) have additional capabilities, illustrated in the following screen, which appear when the option "Use Muso reference database ..." is selected at the top of the tab.

![Genres etc. - Muso](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/genres-muso/)

For these options to work, the path/name of the Muso reference database needs to be specified on the advanced tab. The default path is "C:\\Users\\Public\\Music\\muso\\database" and the default filename is "Reference.xml". The additional options are as follows.

1. "Use Muso classical genres". If this is selected, the box for classical main genres is eliminated and the genre list from Muso's "Tools->Options->Classical Music->Classical Music Genres" is used instead.

2. "Use Muso composer list to determine if classical". If the composer name is in Muso's list "Tools->Options->Classical Music->Composer Roster", then the work will be deemed to be classical. If this option is selected, a further option appears to "Treat arrangers as for composers" - if selected then arrangers will also be looked up in the roster.

3. "Use Muso composer dates (if no work date) to determine period". The birth date + 20  -> death dates of Muso's composer roster will be used to assign periods if no work date is available. If this option is selected, a further option appears to "Treat arrangers as for composers" - if selected then arrangers' working lives will also be used to determine periods.

   (This might be replaced / supplemented by MusicBrainz in  the future, but would involve another 1-second lookup per composer).
   
4. "Use Muso map". Replace the period map with the one in Muso at "Tools->Options->Classical Music->Classical Music Periods"

Note that non-Muso users may also use this functionality, if they wish, by manually creating a reference xml file with the relevant tags, e.g.:

    <ClassicalGenre>  
    <Name>Cantata</Name>  
    </ClassicalGenre>   
    <Composer>  
    <Name>Max REGER</Name>  
    <Birth>1873</Birth>  
    <Death>1916</Death>  
    </Composer>  
    <ClassicalPeriod>  
    <Name>Early Romantic</Name>  
    <Start_x0020_Date>1800</Start_x0020_Date>  
    <End_x0020_Date>1850</End_x0020_Date>  
    </ClassicalPeriod>  

## Tag mapping tab
There are two coloured sections as shown in the screen image below:

![Tag mapping options](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/tag-mapping/)

Note that either the "Create extra artist metadata" option on the Artist tab or "Include all work levels" on the Works tab needs to be selected for these sections to run.

1. "Initial tag processing": This takes place before any of the detailed tag mapping in the second section.

    "Remove Picard-generated tags before applying subsequent actions?". Any tags specified in the next two rows will be blanked before applying the tag sources described in the following section. NB this applies only to Picard-generated tags, not to other tags which might pre-exist on the file: to blank those, use the main Options->Tags page. Comma-separate the tag names within the rows and note that these names are case-sensitive.

    "List existing file tags which will be appended ...": This refers to the tags which already exist on files which have been matched to MusicBrainz, not the tags generated by Picard from the MusicBrainz database. Normally, Picard cannot process these tags - either it will overwrite them (if it creates a similarly named tag), clear them (if 'Clear existing tags' is specified in the main Options->Tags screen) or keep them (if 'Preserve these tags...' is specified after the 'Clear existing tags' option). Classical Extras allows a further option - for the tags to be appended to in the tag mapping section (see below) or otherwise used. List file tags which will be appended to rather than over-written by tag mapping (NB this will keep tags even if "Clear existing tags" is selected on main options). In addition, certain existing tags may be used by Classical Extras - in particular "is\_classical" (which is set by SongKong to '1' if the track is deemed to be classical, based on an extensive database) is used to add 'classical' to the variable "\_cea\_worktype", if "Infer work types" is selected in the first section of the Artists tab. If you include "is\_classical" in this list then any files which have "is\_classical" = 1 will be treated as being classical, regardless of the genre tag.
    
    **Make sure that any tags listed here are not also included in Picard's "Preserve .. tags.." option, otherwise Picard will prevent the tag from being amended**

    Note that if "Split lyrics tag" is specified (see the Artists tab), then the tag named there will be included in the "...existing file tags..." list and does not need to be added in this section.

    "Clear any previous file tags...": This operates in an almost similar way to the main Picard option (Options->Tags->"Clear existing tags"). All existing file tags will be cleared **unless** they are in the main Picard "Preserve tags..." option or the "...existing file tags..." list. The main differences from the basic Picard option are that (a) artwork is always preserved - i.e. this largely addresses [PICARD-257](https://tickets.metabrainz.org/browse/PICARD-257) and (b) the tags that are not kept are **not actually deleted or shown as deleted** in the bottom pane of Picard. They are just not displayed in the bottom pane -  however, a warning tag is written.

2. "Tag map details". This section permits the contents of any hidden variable or tag to be written to one or more tags.

    * **Sources**:
	Some of the most useful sources are available from the drop-down list. Otherwise they can simply be typed in the box. Click on the "source from" button to enable entry (otherwise the text box / drop-down for the source is locked). Some useful names are:

      - soloists : List of performers (with instruments in brackets), who are NOT ensembles or conductors, separated by semi-colons. Note they may not strictly be "soloists" in that they may be part of an ensemble.
      - soloist\_names : Names of the above (i.e. no instruments).
      - vocalists / instrumentalists / other\_soloists : Soloists who are vocalists, instrumentalists or not specified, respectively.
      - vocalist\_names / instrumentalist\_names : Names of vocalists / instrumentalists (i.e. no instrument / voice).
      - ensembles : List of performers who are ensembles (with type / instruments - e.g. "orchestra" - in brackets), separated by semi-colons.
      - ensemble\_names : Names of the above (i.e. no instruments).
      - album\_soloists : Sub-list of soloist\_names who are also album artists.
      - album\_conductors : List of conductors who are also album artists.
      - album\_ensembles: Sub-list of ensemble\_names who are also album artists.
      - album\_composers : List of composers who are also album artists.
      - album\_composer\_lastnames : Last names of composers, of ANY track on the album, who are also album artists. This is the source used to prefix the album name (when that option is selected).
      - support\_performers : Sub-list of soloist\_names who are NOT album artists.
      - composers : List of composers, after applying the naming options in the artists tab.
      - conductors : List of conductors, after applying the naming options in the artists tab.
      - arrangers : Includes all arrangers and instrument arrangers (except more specific roles such as orchestrators) - the standard Picard tag omits some.
      - orchestrators : Arrangers who are orchestrators.
      - leaders : AKA concertmasters.
      - chorusmasters : as distinct from conductors (chorus masters may rehearse the choir but not conduct the performance).

	   Note that the Classical Extras sources for all artist types are spelled in the plural (to differentiate from the native Picard tags). Most of the names are for artist data and are sourced from hidden variables (prefixed with "\_cea\_" or "\_cwp\_"). In specifying the source, the prefix is not necessary - e.g. "arrangers" will pick up all data in \_cea\_arrangers and \_cwp\_arrangers (covering those with recording and work relationships respectively). Using the prefix will get just the specific variable.

	   In addition, the drop-down contains some typical combinations of multiple sources (see note on multiple sources below).

       Any Picard tag names can also be typed in as sources. Any hidden variables may also be used. Any source names which are prefixed by a backslash will be treated as string constants; blanks may also be used.

       It is possible to specify multiple sources. If these are separated by a comma, then each will be appended to the mapped tag(s) (if not already filled or if not "conditional"). So, for example, a source of "album\_soloists, album\_conductors, album\_ensembles" mapped to a tag of "artist" with "conditional" ticked will fill artist (if blanked) by album\_soloists, if any, otherwise album\_conductors etc. Sources separated by a + will be concatenated before being used to fill the mapped tags. The concatenated result **will only be applied if the contents of each of the sources to be concatenated is non-blank** (note that this constraint only applies to **concatenation** of multiple sources). No spaces will be added on concatenation, so these have to be added explicitly by concatenating "\ ".  So, for example "ensemble\_names + \ (conducted by + conductors +\\), ensemble\_names", with "Conditional" selected, will yield something like "BBC Symphony Orchestra (conducted by Walter Weller)" or just "BBC Symphony Orchestra" if there is no conductor. **Do not use any commas in text strings**.

       Another example: to add the leader's name in brackets to the tag with the performing orchestra, put "\\ (leader +leaders+\\)" in the source box and the tag containing the orchestra in the tag box. If there is no leader, the text will not be appended.

     The tag mapping section is not restricted to artist metadata - any metadata or hidden variablescan be used.

    * **Tags**:
	   Enter the (comma-separated) "destination" tag names into which the sources should be written (case sensitive). Note that this will result in the source data being APPENDED in the tag - it will not overwrite the existing contents. Check "Conditional?" if the tag is only to be updated if it is previously blank (all non-empty sources in the current line will be applied in sequence). The lines will be applied in the order shown. Users should be able to achieve most requirements via a combination of blanking tags, using the right source order and "conditional" flags. For example, to overwrite a tag sourced from "composer" with "conductor", specify "conductor" first, then "composer" as conditional. Note that, for example, to demote the MB-supplied artist to only appear if no other listed choices are present, blank the artist tag and then add it as a conditional source at the end of the list.

    * **"Also populate sort tags"**:
     If a sort tag is associated with the source tag then the sort names will be placed in a sort tag corresponding to the destination tag. Note that the only explicit sort tags written by Picard are for artist, albumartist and composer. Picard also writes hidden variables '\_artists\_sort' and 'albumartists\_sort' (note the plurals - these are the sort tags for multi-valued alternatives 'artists' and '\_albumartists'). To be consistent with this approach, the plugin writes hidden variables for other tags - e.g. '\_arranger\_sort'. The plugin also writes hidden sort variables for the various hidden artist variables - e.g. '\_cwp\_librettists' has a matching sort variable '\_cwp\_librettists\_sort'. Therefore most artist-type sources **will** have a sort tag/variable associated with them and these will be placed in a destination sort tag if this option is selected - **in other words, selecting this option will cause most destination tags to have associated sort tags. Furthermore, any hidden sort variables associated with tags which are not listed explicitly in the tag mapping section will also be written out as tags** (i.e. even if the related tags are not included as destination tags). Note, however, that composite sources (e.g. " ensemble\_names + \;  + conductors") do not have sort tags associated with them.

      If this option is not selected, no additional sort tags will be written, but the hidden variables will still be available, so if a sort tag is required explicitly, just map the sort tag directly - e.g. map 'conductors\_sort' to 'conductor\_sort'.

  More complex operations can be built using tagger scripts. If required, these can be set to run conditionally by setting a tag or hidden variable in this section and then testing it in the script.


  
## Advanced tab

Hopefully, this tab should not be much used. In any case, it should not need to be changed frequently. There are seven sections as shown in the sceeen print below:

![Advanced options](https://highmossergate.co.uk/digitalsymphony/classical-extras-screenshots/advanced/)

1. "General". There is only one checkbox - "Do not run Classical Extras for tracks where no pre-existing file is detected (warning tag will be written)". This option will disable Classical Extras processing if no file is present; this means (for example) that single discs from box sets can be loaded without incurring the additional processing overhead (work look-ups etc.) for all the other discs. Also if a compilation album is loaded, where the tracks are on multiple releases, the plugin will only process the release tracks which match. If a file is present but it does not yet have a MusicBrainz trackid tag, then it will initially be treated in the same way as a non-existent file; however, after the initial loading it will (if matched by Picard) be given a MB trackid and "refreshing" the release will result in any such tracks being processed by Classical Extras, while the unmatched tracks are left untouched.

2. "Artists". This has only one subsection - "Ensemble strings" - which permits the listing of strings by which ensembles of different types may be identified. This is used by the plugin to place performer details in the relevant hidden variables and thus make them available for use in the "Tag mapping" tab as sources for any required tags. 
If it is important that only whole words are to be matched, be sure to include a space after the string.

3. "Work levels". This section has parameters applicable to the "works and parts" functions.

	* **Max number of re-tries to access works (in case of server errors)**. Sometimes MB lookups fail. Unfortunately Picard (currently) has no automatic "retry" function. The plugin will attempt to retry for the specified number of attempts. If it still fails, the hidden variable \_cwp\_error will be set with a message; if error logging is checked in section 5, an error message will be written to the log and the contents of \_cwp\_error will be written out to a special tag called "001\_errors" which should appear prominently in the bottom pane of Picard. The problem may be resolved by refreshing, otherwise there may be a problem with the MB database availability. It is unlikely to be a software problem with the plugin.
	
	* **Allow blank part names for arrangements and part recordings, if an arrangement/partial label is provided**. The default is checked (true) - in which case where an arrangement has the same name as the original work, the part will just show the arrangement/partial label (e.g. "Arrangement:"). If the option is blank (false), there will be text for the part name (which may be the same name as the parent work). Note that if "extended" metadata is being used, then blank part names will always have the title metadata added {in curly brackets} regardless of whether it is similar to the canonical part name.

	* **Removal of common text between parent and child works**. This section controls the naming of parts, by stripping parent text from the work name. If the work begins with the parent name followed by punctuation then the common text will always be stripped to give the part name, even if there are punctuation or some other minor differences (synonyms will also be matched using the patterns in the next section). 
	
	  However, common text which is not followed by punctuation or which is not at the start may also be stripped: to prevent this, set "Minimum number of similar words required before eliminating (other than at start)" to zero. Otherwise common text longer than the specified number of words (default = 2) will be stripped. (Note that this minimum is over-ridden by the previous option - i.e. if a smaller number of words than the minimum could be eliminated and would result in a blank part name, and that is allowed by the previous option, then the words will be removed).
	
	* **How title metadata should be included in extended metadata**. This subsection contains various parameters affecting the processing of strings in titles, where these are used to "extend" the text in work names. This is generally only relevant if "Use canonical work metadata enhanced with title text" is selected on the "Works and parts" tab, although the "prefixes" section is also relevant if "Use only metadata from title text" is selected and the synonyms are also used in eliminating parent-child duplication as described above. Because titles are free-form, not all circumstances can be anticipated. If pure canonical works are used ("Use only metadata from canonical works" and, if necessary, "Full MusicBrainz work hierarchy" on the Works and parts tab, section 2) then this processing should largely be irrelevant, but no text from titles will be included. Some explanations are given below:

      * "Proximity of new words". When using extended metadata - i.e. "metadata enhanced with title text", the plugin will attempt to remove similar words in the canonical work name (in MusicBrainz) and the title before extending the canonical name. After removing such words, a rather "bitty" result may occur. To avoid this, any new words with the specified proximity will have the words between them (or up to the start/end) included even if they repeat words in the work name.
      
      * "Treat hyphenated words as two words for comparison purposes" (default = True). In comparing words, hyphenated words will be considered as separate words unless this option is deselected.
      
      * "Proportion of a string to be matched ... for it to be considered essentially similar..." (default = 66%). If the title and work descriptions are largely the same then the title text will not be used to extend the work text, even if there are some new words.
      
      * "Prepositions and conjunctions". Words listed here will not generally be treated as "new" (i.e. if they are in the title text but not in the work text, they will not be included in the "extended" text) unless they precede a new word which is not itself a preposition. Note that, although the term "preposition" is used here, because that is the obvious usage, any word can be listed. A group of more than one such words will be treated as new if they are not in the work text and they precede a new word which is not listed as a preposition.

      * "Prefixes". When using "metadata from titles" or extended metadata, the structure of the works in MusicBrainz is used to infer the structure in the title text, so strings that are repeated between tracks which are part of the same MusicBrainz work will be treated as "higher level". This can lead to anomalies if, for instance, the titles are "Work name: Part 1", "Work name: Part 2", "Part" is repeated and so will be treated as part of the parent work name. Specifying such words in "Prefixes" will prevent this.

      * "Synonyms". These words/phrases will be considered equivalent when comparing work name and title text  (or parent and child text). Thus if one word/phrase appears in the work name, it and its synonyms will be removed from the title in extending the metadata (subject to the proximity setting above). Each entry should be a tuple in the form *(key word/phrase, ...,  equivalent word/phrase)* - no quote marks are necessary. Each tuple should be separated by a forward slash - /. Tuples may contain multiple synonyms - each item in a tuple will be treated as similar when comparing works/parts and titles. The text in tags will be unaltered. Spaces and punctuation are permitted, as are regular expressions, but unless entering regex, use backslash \ to escape any regex metacharacters \ ^ $ . | ? * + ( ) [ ] {   
      Also escape commas , and forward slashes /. Do not enclose strings in quote marks. The last member of the tuple should be the canonical form (i.e. the one to which others are converted for comparison) and must be a normal string (not a regex). The sequence of synonyms within a tuple may be important, particularly if one synonym is a subset of another - always list the longer one first, as the matching will proceed in the listed order. Thus if "nro" and "nr" are synonyms of "no", then list them thus: (nro, nr, no). This will ensure that "nro" gets matched to "nro" and not to "nr".

      * "Replacements". These are entered in a similar fashion to synonyms. The difference is that the last item in a tuple will be used to replace any text in earlier items in the tuple. This last element may also be left blank (after the preceding comma) in order to remove any text matching any of the earlier items.

4. "Genres etc. ...". This is only required if Muso-specific options are used for genres/periods. Specify the path and file name for the reference database (the default is the Muso default for a shared  database). Note that the database is only loaded when Picard starts so you will need to restart Picard is these options are changed.

5. "Logging options". These options are in addition to the options chosen in Picard's "Help->View error/debug log" settings. They only affect messages written by this plugin. To enable debug messages to be shown in the Picard log, the flag needs to be set here and "Debug mode" needs to be turned on in the log. **It is strongly advised to keep the "debug" flag unchecked unless debugging is required** as it slows up processing and may even cause Picard to hang if there is a large number of files (better to use the 'full' custom logging option - see below). The "error" and "warning" flags should be left checked, unless it is required to suppress messages (the messages are also written to the tags 001\_errors and 002\_warnings).

   As well as the main Picard log, a custom logging function is provided. This may be either "basic" or "full". If "basic" is selected, a file "session.log" will be written (over-written each session) to a "Classical Extras" directory inside the same directory as the plugins folder. (Note - the easy way to find this directory is to select options-->plugins in Picard and the "Open plugins folder" button, then go up one level). The session log gives a processing summary for each release and includes errors, warnings and debug messages if those options have been selected.

   If "full" is selected, all errors, warnings and debugs will be written, along with additional debugging messages, to a custom log file for each release processed. These files are stored in the "Classical Extras" directory inside the same directory as the plugins folder. The log file for a release is named using the release MBID. Debugging from these files requires an understanding of the source code.
   
   Selecting "full" will slow Picard, but should not normally result in hanging or crashing.

6. "Save plugin details and options in a tag?" can be used so that the user has a record of the version of Classical Extras which generated the tags and which options were selected to achieve the resulting tags. Note that the tags will be blanked first so this will only show the last options used on a particular file. The same tag can be used for both sets of options, resulting in a multi-valued tag. All the options in the Classical Extras UI are saved **except** those which are asterisked.

   The tag contents are in dict format. The options in these tags can then be used to over-ride the displayed options subsequently (see below).

   N.B. The "Tag name for artist/misc. options" also saves the Picard options for 'translate\_artist\_names' and 'standardize\_artists' as these interact with the Classical Extras options.

7. "Over-ride plugin options displayed in UI with options from local file tags". If options have previously been saved (see above), selecting these will cause the saved options to be used in preference to the displayed options. The displayed options will not be affected and will be used if no saved options are present. The default is for no over-ride.

   ***Note that* *very occasionally (if the tag containing the options has been corrupted) use of this option may cause an error. In such a case you will need to deselect the "over-ride" option and set the required options manually; then save the resulting tags and the corrupted tag should be over-written***

   The last checkbox, "Overwrite options in Options Pages", is for **VERY CAREFUL USE ONLY**. It will cause any options read from the saved tags to over-write the options on the plugin Options Page UI. (Note that it will only operate if all the "Over-ride plugin options..." boxes are checked as well.) The intended use of this is if for some reason the user's preferred options have been erased/reverted to default - by using this option, the previously-used choices from a reliable filed album can be used to populate the Options Page. The box will automatically be unticked after loading/refreshing one album, and will always be turned off when starting Picard, to prevent inadvertant use. Far better is to make a **backup copy** of the picard.ini file.

# Information on hidden variables

This section is for users who want to write their own scripts, or add additional tags (in the tag mapping section) based on hidden variables. The definition and source of each hidden variable is listed. Apologies if there are errors and omissions in this section - to double check the actual hidden variables for any track, use the plugin "View script variables".

## Works and parts

- \_cwp\_work\_n, where n is an integer >=0 : The MB work name at level n. For n=0, the tag is the same as the current standard Picard tag "work"
- \_cwp\_work\_top : The top work name (i.e. for maximal n). Thus, if max n = N, \_cwp\_work\_top will be equivalent to \_cwp\_work\_N. Note, however, that this will always be the "canonical" MB name, not one derived from titles or the lowest level work name and that no annotations (e.g. key or work year) will be added (whereas they will be added to \_cwp\_work\_N). Nevertheless, if "replace work names by aliases" has been selected and is applicable, the relevant alias will be used.
- \_cwp\_workid\_n : The matching work id for each work name. For n=0, the tag is the same as the standard Picard tag "MusicBrainz Work Id"
- \_cwp\_workid\_top : The matching work id for the top work name.
- \_cwp\_part\_n : A "stripped" version of \_cwp\_work\_n, where higher-level work text has been removed wherever possible, to avoid duplication on display.
	Thus in theory, \_cwp\_work\_0 will be the same as "\_cwp\_work\_top: \_cwp\_part\_(N-1): ...: \_cwp\_part\_0" (punctuation excepted), but may differ in more complex situations where there is not an exact hierarchy of text as the work levels are traversed. (See below for the "\_X0" series which attempts to address any such inconsistencies)
- \_cwp\_part\_levels : The number of work levels attached to THIS TRACK. Should be equal to N = max(n) referred to above.
- \_cwp\_work\_part\_levels : The maximum number of levels for ANY TRACK in the album which has the same top work as this track.
- \_cwp\_single\_work\_album : A flag = 1 if there is only one top work in this album, else = 0.?
- \_cwp\_work : the level selected by the plugin to be the source of the single-level work name if "Use only metadata from canonical works" is selected (usually the top level, but one lower in the case of a single work album).
- \_cwp\_groupheading : the level selected by the plugin to be the source of the multi-level work name if "Use only metadata from canonical works" is selected.
- \_cwp\_part : The movement name derived from the MB work names (generally = \_cwp\_part\_0) and used as the source for the movement name used for "Tags for Movement - including embedded movt/part numbers".
- \_cwp\_inter\_work : Intermediate works between \_cwp\_part and \_cwp\_work (if any).

If there is more than one work any level, then \_cwp\_work\_n and \_cwp\_workid\_n will have multiple entries. Another common situation is that a "bottom level" work is spread across more than one track. Rather than artificially split the work into sub-parts, this is often shown in MusicBrainz as a track being a "partial recording of" a work. The plugin deals with this by creating a notional lowest-level with the suffix " (part)" (or other text as defined in the works and parts options tab) appended to the work it is a partial recording of. In order that this notional part can be separately identified from the full work, the musicbrainz\_recordingid is used as the identifier rather than the workid.
If there is more than one "parent" work of a lower level work, multi-valued tags are generated.

- \_cwp\_X0\_part\_0 : A "stripped" version of \_cwp\_work\_0 (see above), where elements of \_cwp\_work\_0 which repeat within level 1 have been stripped.
- \_cwp\_X0\_work\_n : The elements of \_cwp\_work\_0 which repeat within level n

As well as variables derived from MB's work structure, some variables are produced which are derived from the track title. Typically titles may be in the format "Work: Movement", but not always. Sometimes the title is prefixed by the name of the composer; in this case the variable
- \_cwp\_title
is provided which excludes the composer name and subsequent processing is carried out using this rather than the full title. 

The plugin uses a number of methods attempt to extract the works and movement from the title. The resulting variables are:
- \_cwp\_title\_work\_n, and
- \_cwp\_title\_part\_n
which mirror those for the ones based on MB works described above.
- \_cwp\_title\_part\_levels which similarly mirrors \_cwp\_part\_levels
- \_cwp\_title\_work\_levels which similarly mirrors \_cwp\_work\_part\_levels

- \_cwp\_title\_work is the level selected by the plugin to be the source of the single-level work name if "Use only metadata from title text" is selected (usually the top level, but one lower in the case of a single work album).
- \_cwp\_title\_groupheading is similarly the level selected by the plugin to be the source of the multi-level work name if "Use only metadata from title text" is selected.

- \_cwp\_extended\_part : = \_cwp\_part with additional movement information from the title - given in {}.
- \_cwp\_extended\_groupheading : = \_cwp\_groupheading with additional work information from the title - given in {}.
- \_cwp\_extended\_work : = \_cwp\_work with additional work information from the title - given in {}.
- \_cwp\_extended\_inter\_work : = \_cwp\_inter\_work with additional work information from the title - given in {}.
The "extended" variables can be useful where the "canonical" work names in MB are in the original language and the titles are in English (say). Various heuristics are used to try and add (and only add) meaningful additional information, but oddities may occur which require manual editing.

Artist tags which derive from work-artist relationships are also set in this section:
- \_cwp\_composers & \_cwp_composers_sort
- \_cwp\_composer_lastnames
- \_cwp\_writers & \_cwp_writers_sort
- \_cwp\_arrangers : This is for arrangers of the work and also "instrument arrangers" and "vocal arrangers" with appropriate annotation for instrument and voice types. (Picard does not currently write the latter to the Arranger tag if they are part of the work-artists relationship, despite style guidance saying to use specific instrument types instead of generic arranger.) 
- \_cwp\_arranger\_names & \_cwp_arrangers_sort: Just the names of the above (no annotations)
- \_cwp\_orchestrators & \_cwp_orchestrators_sort
- \_cwp\_reconstructors & \_cwp_reconstructors _sort - 'reconstructed by' relationships
- \_cwp\_revisors & \_cwp_revisors _sort - 'revised by' relationships
- \_cwp\_lyricists & \_cwp_lyricists _sort
- \_cwp\_librettists & \_cwp_librettists _sort
- \_cwp\_translators & \_cwp_translators _sort

Finally, the tags \_cwp\_error and\_cwp\_warning are provided to supply warnings and error messages to the user.

## Artists

All the additional hidden variables for artists written by Classical Extras are prefixed by \_cea\_. Note that these are generally in the plural, whereas the standard tags are singular. If the user blanks a tag then the original value is stored in the singular with the \_cea\_ prefix. Thus \_cea\_arranger would be the contents of the Picard tag "arranger" before blanking, whereas \_cea\_arrangers is hidden variable created by Classical Extras.

- \_cea\_recording\_artist : The artist credited with the recording (not necessarily the track artist). Note that this is the only "\_cea\_" tag which is singular, because it is in the same format as the 'artist' tag, whereas...
- \_cea\_recording\_artists : The list/multiple value version of the above. (This follows the approach in Picard for 'artist' and 'artists', being the track artists.)
- \_cea\_MB\_artists: The original track artists per MusicBrainz before any replacement by / merging with recording artists.
- \_cea\_soloists : List of performers (with instruments in brackets), who are NOT ensembles or conductors, separated by semi-colons. Note they may not strictly be "soloists" in that they may be part of an ensemble.
- \_cea\_recording\_artistsort : Sort names of \_cea\_recording\_artist
- \_cea\_recording\_artists\_sort : Sort names of \_cea\_recording\_artists
- \_cea\_soloist\_names : Names of the soloists (i.e. no instruments).
- \_cea\_soloists\_sort : Sort\_names of the above.
- \_cea\_vocalists : Soloists who are vocalists (with voice in brackets).
- \_cea\_vocalist\_names : Names of the above (no voice).
- \_cea\_instrumentalists : Soloists who have instruments but are not vocalists.
- \_cea\_instrumentalist\_names : Names of the above (no instrument).
- \_cea\_other\_soloists : Soloists who do not have specified instrument/voice.
- \_cea\_ensembles : List of performers which are ensembles (with type / instruments - e.g. "orchestra" - in brackets), separated by semi-colons.
- \_cea\_ensemble\_names : Names of the above (i.e. no instruments).
- \_cea\_ensembles\_sort : Sort\_names of the above.
- \_cea\_album\_soloists : Sub-list of soloist\_names who are also album artists
- \_cea\_album\_soloists\_sort : Sort\_names of the above.
- \_cea\_album\_conductors : List of conductors who are also album artists
- \_cea\_album\_conductors\_sort : Sort\_names of the above.
- \_cea\_album\_ensembles: Sub-list of ensemble\_names who are also album artists
- \_cea\_album\_ensembles\_sort : Sort\_names of the above.
- \_cea\_album\_composers : List of composers who are also album artists
- \_cea\_album\_composers\_sort : Sort\_names of the above.
- \_cea\_album\_track\_composer\_lastnames : Last names of the above. (N.B. This only includes the composers of the current track - compare with \_cea\_album\_composer\_lastnames below).
- \_cea\_album\_composer\_lastnames : Last names of composers of ANY track on the album who are also album artists. This can be used to prefix the album name if required. (cf \_cea\_album\_track\_composer\_lastnames)
- \_cea\_support\_performers : Sub-list of soloist\_names who are NOT album artists
- \_cea\_support\_performers\_sort : Sort\_names of the above.
- \_cea\_composers : Alternative to 'composer', incorporating 'naming options' on the artists tab.
- \_cea\_composer_lastnames: Last names of above.
- \_cea\_conductors : Alternative to 'conductor', incorporating 'naming options' on the artists tab.
- \_cea\_performers : Alternative to 'performer', incorporating 'naming options' on the artists tab.
- \_cea\_arrangers : All arrangers for the **recording** with instrument/voice type in brackets, if provided. If the work and parts functionality has also been selected, the arrangers of works, which Picard also currently omits will be put in \_cwp\_arrangers.
- \_cea\_orchestrators : Arrangers (per Picard) included in the MB database as type "orchestrator".
- \_cea\_chorusmasters : A person who (per Picard) is a conductor, but is "chorus master" in the MB database (i.e. not necessarily conducting the performance).
- \_cea\_leaders : The leader of the orchestra ("concertmaster" in MusicBrainz) - not created by Picard as standard. 

## Genres etc.

Most of the genres, keys and date information requires the works and parts section to have been run, and so the variables are prefixed \_cwp\_, but instruments and genres which are derived from artist information and are prefixed \_cea\_.

- \_cea\_instruments : Names of all instruments on the track (MusicBrainz names)
- \_cea\_instuments\_credited : As above, but MB names replaced by as-credited names, if any
- \_cea\_instruments\_all : MB and as-credited names
- \_cea\_work\_type : The genre(s) inferred from artist information
- \_cea\_work\_type\_if\_classical : As above, but only of relevance if the work is classical
- \_cwp\_candidate\_genres : List of all tags, work types etc. found (depending on specified sources) before filtering for "allowed" genres.
- \_cwp\_keys : keys associated with this track (from all work levels).
- \_cwp\_composed\_dates : Date composed (integer) or range (integer-integer).
- \_cwp\_published\_dates : Date published (integer) or range (integer-integer).
- \_cwp\_premiered\_dates : Date premiered (integer) or range (integer-integer).
- \_cwp\_untagged\_genres : Genres in \_cwp\_candidate\_genres which have been filtered out as they are not in any "allowed" list.
- \_cwp\_unrostered\_composers : For Muso users: composers who are not in Muso's classical composers roster.

# Software-specific notes

Note that \_cwp\_part\_levels > 0 will indicate that the track recording is part of a work and so could be used to set other software-specific flags (e.g. for iTunes "show work movement") to indicate a multi-level "work: movement".

## SongKong

SongKong users may wish to map the \_cwp variables to tags produced by SongKong if consistency is desired, in which case the mappings are principally:
- \_cwp\_work\_0 => musicbrainz\_work\_composition
- \_cwp\_workid\_0 => musicbrainz\_work\_composition\_id
- \_cwp\_work\_n => musicbrainz\_work\_part\_leveln, for n = 1..6
- \_cwp\_workid\_n => musicbrainz\_work\_part\_leveln\_id, for n = 1..6
- \_cwp\_work\_top => musicbrainz\_work

These mappings are carried out automatically if the relevant options on the "Work and parts" tab are selected.  
In addition, \_cwp\_title\_work and \_cwp\_title\_part\_0 are intended to be equivalent to SongKong's work and part tags.
(N.B. Full consistency between SongKong and Picard may also require the modification of Artist and related tags via a script, or the preservation of the related file tags)

## Muso

The tag "groupheading" should be set as the "Tags for Work - for software with 2-level capability". Muso will use this directly and extract the levels from it (split by the double colon). Muso permits a variety of import options which should be capable of combination with the tagging options in this plugin to achieve most desired effects. To avoid the use of import options in Muso, set the output tags from the plugin to be the native ones used by Muso (NB "title" may include or exclude groupheading - Muso should recognise it and extract it).

To make use of Muso's in-built classical music processing to set explicit tags in Picard, enable the "Use Muso reference database ..." option on the "Genres etc." tab.

## Players with no "work" capability

Leave the "title" tag unchanged or make it a combination of work and movement.

# Possible Enhancements
All planned functionality has been included in version 2.0. Please post any suggestions for improvements to the forum.

# Technical Matters (of possible interest to developers)
Issues were encountered with the Picard API in that there is not a documented way to let Picard know that it is still doing asynchronous tasks in the background and has not finished processing metadata. Many thanks to @dns\_server for assistance in dealing with this and to @sophist for the albumartist\_website code which I have borrowed from. I have tried to add some more comments to help any others trying the same techniques.

Also, the documentation of the XML lookup is virtually non-existent. The response is an XmlNode object (not a dict, although it is represented as one). Each node has a name with {attribs, text, children} values. The structure is more clearly understood if the web-based lookup is used (which is well documented at https://musicbrainz.org/doc/Development/XML_Web_Service/Version_2 ) as this gives an XML response. I wrote a function (parse\_data) to parse XmlNode objects, or lists thereof, for a (parameterised) hierarchy of nodes (and optional attribs value tests) in order to extract required data from the response. This may be of use to other plugin authors. The situation has been made slightly better in Picard 2.0 as the objects returned from register_track_metadata_processor are standard JSON. See https://musicbrainz.org/doc/Development/JSON_Web_Service. My parse_data function works with JSON as well as XmlNode formats, but the arguments you need to provide to it are different as the formats are structured differently. Picard 2.0 allows tagger.webservice calls to specify XML or default to JSON. This plugin has now migrated over to JSON-only.

To get the whole picture, in XML, for a release, use (for example) https://musicbrainz.org/ws/2/release/f3bb4fdd-5db0-43a8-be73-7a1747f6c2ef?inc=release-groups+media+recordings+artist-credits+artists+aliases+labels+isrcs+collections+artist-rels+release-rels+url-rels+recording-rels+work-rels+recording-level-rels+work-level-rels. (Add &fmt=JSON at the end to get the JSON response). This simulates the response given by Picard with the "Use release relationships" and "Use track relationships" options selected. Note that the Picard album\_metadata\_processor returns releaseXmlNode (as JSON from Picard 2.0 on) which is everything from the Release node of the XML downwards, whereas track\_metadata\_processor returns trackXmlNode which is everything from a Track node downwards (release -> medium-list -> medium -> track-list is above track). Replace all hyphens in XML with underscores when parsing the Python object (JSON uses hyphens).

Care needs to be taken when using the metadata property in Picard (e.g. as in track.metadata). This returns an object of class Metadata, which looks just like a dictionary but isn't. All values are in fact lists, which works as follows: when a value is set, it is always set as a list - i.e. a list value is entered as it is, but a string  is set as a list : viz. ['string']. Then, when getting an item from the object, multiple values are joined to create a string. Thus track.metadata['item'] and track.metadata.get('item') will return list.join('; ') - i.e. a string with the list values separated by semi-colons - not the list. To get the list, you need to use track.metadata.getall('item'). The plugin stores work ids, internally and in "hidden variables" (such as _cwp_workid_0), as tuples. The Metadata class puts these into a string before setting the value in the object as a list, so the resulting value is ["(wid1, wid2, ...)"]. In order to extract the original tuple, the eval function is required.

A large variety of releases were used to test this plugin, but there may still be bugs, so further testing is welcome. The following release was particularly complex and useful for testing: https://musicbrainz.org/release/ec519fde-94ee-4812-9717-659d91be11d4. Also this release was a bit tricky - a large box set with some works appearing as originals and in arrangements: https://musicbrainz.org/release/5288f266-bab8-45bd-83e4-555730f02fa0.

# Very technical matters (probably not of interest to anyone)
I've done a bit of research and observed the following behaviour in Picard when using the  `register_track_metadata_processor()` API:
1. If a new album (i.e. not yet tagged by Picard and with no MBIDs) is loaded, clustered and looked-up/scanned, resulting in the matched files being shown in the right-hand pane, then:
    * `track.metadata` gives the lookup result from MB - i.e. no file information, just the track info.
    * `album.tagger.files[xxxx].metadata` (where xxxx is the path/filename of the track) gives the file information (dirname etc.) and the tags on the original file.
    * `album.tagger.files[xxxx].orig_metadata` gives the same as `album.tagger.files[xxxx].metadata`

2. However, if the album is then "refreshed", this does not just carry out a repeat operation, instead:
    * `track.metadata` gives the same as before
    * `album.tagger.files[xxxx].metadata` gives the metadata as in `track.metadata` and also includes all the file information as before.
    * `album.tagger.files[xxxx].orig_metadata` gives the same as before (i.e. the original tags).

So, the 'post-refreshment' metadata is actually usable - by matching the `musicbrainz_trackid` of `track.metadata` and `album.tagger.files[xxxx].metadata`, you can get the file details of the track. Not elegant, but it works.
But why is it necessary to "refresh" to get what seems like a logical data set? Basically, the metadata process is not complete when the plugin is called, so not all metadata is available to it. 

I don't know why `get_files_from_objects(objs)` doesn't work in the `register_track_metadata_processor()` API when `objs` is a list of tracks, but does provide a list of file objects when `objs` is a list of albums. Also it does work in the `register_file_action(xxx)`/`register_track_action(xxx)` API, but I assume that is because `itemsview.py` can identify that you have clicked on an item that is a track and a file (this is after the metadata processing has run).

In order to get round these problems, I have adopted a 'hack' which looks up the files for an album and finds which file matches the required track on tracknumber and discnumber. This seems to work, but there may be circumstances when it does not. As a consequence, refreshment should only be required if `get_files_from_objects(objs)` doesn't get all the album files.

# List of previous updates

Version 0.9.4: Bug fixes. The last released version for Picard 1.4 and the fork for version 2.0.

Version 0.9.3: Custom logging if enabled on "Advanced" tab - writes one log file per album in addition to standard Picard log. Also a session log with basic data is written. Performance improvements (especially for lyrics processing) and various bug fixes.

Version 0.9.2: A new tab in the options page has been created - "Genres etc." This includes special processing  for genres, instruments, keys, work dates and periods. The "Infer work types" option on the "Artists" tab has been moved to this tab (but will be reset to the default of "False" and will need to be reset as required, because it operates slightly differently now that there is more control over the setting of genres). A separate section has ben added to this readme giving more details on the options in this tab.

Version 0.9.1: Bug fixes.

Version 0.9: Additional option to clear previous file tags which are not wanted, without interfering with cover art. Additional option to replace instrument names with as-credited names. Also instrument names are now saved to hidden variables tags (instruments, instruments\_credited and instruments\_all) which can be mapped to file tags as required. Sub-option added to the 'override artist options' option on the "advanced" tab - to allow tag map details to be included or not in this over-ride. Minor bug fixes and UI improvements. This is the next 'official' version after 0.7.

Version 0.8.9: Provide option (in advanced tab) to disable Classical Extras processing if no file is present; this enables (for example) single discs from box sets to be loaded without incurring the additional processing overhead for all the other discs. The settings of the main Picard options "translate\_artist\_names" and "standardize\_artists" is now saved along with the Classical Extras options so that they can be used to over-ride the displayed options. This is because they interact with the Classical Extras options in certain cases. Also:- graceful recovery from authentication failure; improved UI - more scalable; minor bug fixes.

Version 0.8.8: Fixes to allow for (1) disabling of 'use\_cache' across releases which may share works and (2) works which might appear in their own right and also have arrangements. Also, re-set certain important options to defaults on starting Picard, namely: 'use\_cache' set to True, 'log\_debug', 'log\_info' and 'options\_overwrite' set to False; the user will need to deliberately re-set these on starting Picard if required - this is to prevent inadvertently leaving these flags in an abnormal state.

Version 0.8.7: Revised treatment of "conditional" tag mapping. Previously, if multiple sources were specified for a tag mapping and the "conditional" flag set, only the first non-empty source was used. Now all sources will be mapped to a tag if it was empty before executing the current tag mapping line. This is considered to be more intuitive and leads to less complex mapping lines. However, it some cases it may be necessary to split a line from a previous version if the previous behaviour was specifically desired. Improved algorithms for extending metadata with title info. Bug fixes.

Version 0.8.6: More consistent approach to sort tags and hidden variables. Bug fixes.

Version 0.8.5: Improved handling of instruments. Bug fixes.

Version 0.8.4: Improved UI, bug fixes and code improvements.

Version 0.8.3: Ability to use aliases of work names instead of the MusicBrainz standard names. Various options to use aliases and as-credited names for performers and writers (i.e. relationship artists). Option to use recording artists as well as (or instead of) track artists. All relationship artists are now tagged for all work levels (with some special processing for lyricists). New UI layout to separate tag mapping from artist options. Option to split lyrics into common (release) and unique (track) components. Various code improvements as well as bug fixes.

Version 0.8.2: Improved algorithms and a few minor bug fixes.

Version 0.8.1: Bug fixes and minor enhancements - including revison and extension of "synonyms" section on the "advanced" tab.

Version 0.8: Handle multiple recordings and/or multiple parents of a work. 
Handle multiple albumartist composers for one track. 
Option to use "credited as" name for artists (inc. performers and composers) who are "release artist" or "track artist". 
Option to exclude "solo" from instrument types. 
Option to over-ride plugin options with those used when the album was last saved. 
Option to keep (and append to) specified existing file tags - furthermore if "is\_classical" is present, the work-type variable will include "Classical". 
Option to use (and write) SongKong-compatible work tags (saves processing time if SongKong is used to pre-process large numbers of files). 
Include the work (and its parents) of which a work is an arrangement (as a "pseudo-parent"). 
Include medleys in movement/part description (as [Medley of:...] or other descriptor specified in options). 
Allow for multiple parents of recordings and works (and multiple parents of those) - multiples are given as multiple tag instances, where permitted, otherwise separated by semi-colons. 
Option as to whether to include parent works where the relationship attribute is "part of collection". 
Plus minor enhancements and bug fixes too numerous to mention!
(Note that some old versions of the plugin may say v0.8 when they are only v0.7)

Version 0.7: Bug fixes. Pull request issued for this version.

Version 0.6.6: Bug fixes and algorithm improvements. Allow multiple sources for a tag (each will be appended) and use + as separator for concantenating sources, not a comma, to distinguish from the use of comma to separate multiples. Provide additional hidden variables ("sources") for vocalists and instrumentalists. Option to include intermediate work levels in a movement tag (for systems which cannot display multiple work levels).

Version 0.6.5: Include ability to use multiple (concatenated) sources in tag mapping (see notes under "tag mapping"). All artist "sources" using hidden variables (\_cea\_...) are now consistently in the plural, to distinguish from standard tags. Note that if "album" is used as a source in tag mapping, it will now be the revised name, where composer prefixing is used. To use the original name, use "release" as the source. Also various bug fixes, notably to ensure that all arrangers get picked up for use in tag mapping.

Version 0.6.4: Write out version number to user-specified tag. Provide default comment tags for writing ui options. Provide better m4a and iTunes compatibility (see notes). Added functionality for chorus master, orchestrator and concert master (leader). Re-arranged artist tab in ui. Various bug fixes.

Version 0.6.3: Bug fixes. Modified ui default options.

Version 0.6.2: Bug fixes. More flexible handling of artists (can blank and then add back later). Modified ui default options.

Version 0.6.1: Amended regex to permit non-Latin characters in work text.