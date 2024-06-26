# GDIMetaFile class

The `GDIMetaFile` class reads, writes, manipulates and replays metafiles via the
`VCL` module.

A typical use case is to initialize a new `GDIMetaFile`, open the actual stored
metafile and read it in via `GDIMetaFile::Read( aIStream )`. This reads in the
metafile into the `GDIMetafile` object - it can read in an old-style `VCLMTF`
metafile (back in the days that Microsoft didn't document the metafile format
this was used), as well as EMF+ files - and adds them to a list (vector) of
`MetaActions`. You can also populate your own `GDIMetaFile` via `AddAction()`,
`RemoveAction()`, `ReplaceAction()`, etc.

Once the `GDIMetafile` object is read to be used, you can "play" the metafile,
"pause" it, "wind forward" or "rewind" the metafile. The metafile can be moved,
scaled, rotated and clipped, as well have the colours adjusted or replaced, or
even made monochrome.

The GDIMetafile can be used to get an `OutputDevice`'s metafile via the
`Linker()` and `Record()` functions.

## Using GDIMetafile

First, create a new `GDIMetafile`, this can be done via the default constructor.
It can then be constructed manually, or you can use `Record()` on an
`OutputDevice` to populate the `GDIMetaFile`, or of course you can read it from
file with `Read()`. From here you can then elect to manipulate the metafile, or
play it back to another `GDIMetafile` or to an `OutputDevice` via `Play()`. To
store the file, use `Write()`.

### CONSTRUCTORS AND DESTRUCTORS

- `GDIMetaFile`
- `GDIMetaFile( const GDIMetaFile& rMtf )` - copy constructor
- `~GDIMetaFile`

### OPERATORS

- `operator =`
- `operator ==`
- `operator !=`

### RECORDING AND PLAYBACK FUNCTIONS

- `Play(GDIMetaFile&, size_t)`               - play back metafile into another
                                             metafile up to position
- `Play(OutputDevice*, size_t)`              - play back metafile into
                                             `OutputDevice` up to position
- `Play(OutputDevice*, Point, Size, size_t)` - play back metafile into
                                             `OutputDevice` at a particular
                                             location on the `OutputDevice`, up
                                             to the position in the metafile
- `Pause`                                    - pauses or continues the playback
- `IsPause`
- `Stop`                                     - stop playback fully
- `WindStart`                                - windback to start of the metafile
- `windPrev`                                 - windback one record
- `GetActionSize`                            - get the number of records in the
                                             metafile


### METAFILE RECORD FUNCTIONS

- `FirstAction`                              - get the first metafile record
- `NextAction`                               - get the next metafile record from
                                             the current position
- `GetAction(size_t)`                        - get the metafile record at
                                             location in file
- `GetCurAction`                             - get the current metafile record
- `AddAction(MetaAction*)`                   - appends a metafile record
- `AddAction(MetaAction*, size_t)`           - adds a metafile record to a
                                             particular location in the file
- `RemoveAction`                             - removes record at file location
- `Clear`                                    - first stops if recording, then
                                             removes all metafile records
- `push_back`                                - pushes back, basically a thin
                                             wrapper to the metafile record list


### READ AND WRITING

- `Read`
- `Write`
- `GetChecksum`
- `GetSizeBytes`


### DISPLACEMENT FUNCTIONS

- `Move( long nX, long nX)`
- `Move( long nX, long nX, long nDPIX, long nDPIY )` - Move method getting
                                             specifics how to handle
                                             MapMode( MapUnit::MapPixel )


### TRANSFORMATION FUNCTIONS

- `Scale( double fScaleX, double fScaleY )`
- `Scale( const Fraction& rScaleX, const Fraction& rScaleY )`
- `Mirror`
- `Rotate( long nAngle10 )`
- `Clip( const Rectangle& )`


### COLOR ADJUSTMENT FUNCTIONS

- `Adjust`                                    - change luminance, contrast,
                                              gamma and RGB via a percentage
- `Convert`                                   - colour conversion
- `ReplaceColors`
- `GetMonochromeMtf`

## Related classes

`MetaAction`: a base class used by all records. It implements a command-like
pattern, and also acts as a prototype for other actions.

### CONSTRUCTORS AND DESTRUCTORS

- `MetaAction()`                              - default constructor, sets mnRefCount to 1 and
                                              mnType, in this case MetaActionType::NONE
- `MetaAction(sal_uInt16 nType)`              - virtual constructor, sets mnType to nType, and
                                              mnRefCount to 1
- `~MetaAction`

### COMMAND FUNCTION

- `Execute(OutputDevice*)`                    - execute the functionality of the record to the
                                              OutputDevice. Part of command pattern. 
### FACTORY FUNCTION

- `Clone()`                                   - prototype clone function

### MANIPULATION FUNCTIONS

- `Move(long nHorzMove, long nVerMove)`
- `Scale(double fScaleX, double fScaleY)`


### READ AND WRITE FUNCTIONS

- `Read`
- `Write`
- `ReadMetaAction`                            - a static function, only used to determine which
                                              MetaAction to call on to read the record, which
                                              means that this is the function that must be used.

### INTROSPECTIVE FUNCTIONS

- `GetType`

### A note about MetaCommentAction

So this class is the most interesting - a comment record is what is used to
extended metafiles, to make what we call an "Enhanced Metafile". This basically
gets the `OutputDevice`'s connect metafile and adds the record via this when it
runs `Execute()`. It doesn't actually do anything else, unlike other
`MetaAction`s which invoke functions from `OutputDevice`. And if there is no
connect metafile in `OutputDevice`, then it just does nothing at all in Execute.
Everything else works as normal (Read, Write, etc).

## Basic pseudocode

The following illustrates an exceptionally basic and incomplete implementation
of how to use `GDIMetafile`. An example can be found at
`vcl/workben/mtfdemo.cxx`

```
DemoWin::Paint()
{
    // assume that VCL has been initialized and a new application created

    Window* pWin = new WorkWindow();
    GDIMetaFile* pMtf = new GDIMetaFile();

    SvFileStream aFileStream("example.emf", STEAM_READ);

    ReadWindowMetafile(aFileStream, pMtf);
    pMtf->Play(pWin);
    
}
```
