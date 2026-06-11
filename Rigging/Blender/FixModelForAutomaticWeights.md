
In Edit Mode on your mesh:

Remove doubles/merge vertices: Select All → M → Merge by Distance
Fix normals: Select All → Mesh > Normals > Recalculate Outside (Shift+N)
Delete loose geometry: Mesh > Clean Up > Delete Loose
Fill holes: Select border edges → F to fill, or Mesh > Clean Up > Fill Holes






# Joining & Merging Objects in Blender
There are two levels to this depending on what you want:

Step 1: Join the Objects
In Object Mode:

Select all the objects you want to combine (Shift+click each, or A for all)
Make sure the object you want as the "master" (it inherits the name/origin) is the active object (brighter outline) — Ctrl+click it last
Press Ctrl+J to join them into one object

This merges them into a single mesh object, but the geometry is still separate islands inside it.

Step 2: Merge the Geometry (if needed)
If you need the meshes to be truly connected (shared vertices, no seams), go into Edit Mode after joining:
Remove overlapping/doubled vertices:

Select All (A)
M → Merge by Distance (adjust the distance threshold in the bottom-left popup if needed)

Manually weld specific vertices:

Select the two vertices you want to merge
M → At Center or At Cursor


Step 3: Fix the Result
After joining, clean things up:

Recalculate normals: Select All → Shift+N (fixes inside-out faces)
Check for non-manifold geometry: Select > Select All by Trait > Non-Manifold — fix any flagged areas
Apply transforms: Back in Object Mode, Ctrl+A → All Transforms
Set a clean origin: Right-click > Set Origin > Origin to Geometry (or to center of mass)


When to Use Boolean Instead
If your objects overlap and you want them to actually cut into or fuse with each other (not just share vertices at the seams), use a Boolean modifier instead of just joining:

Select the main object
Add a Boolean modifier (Properties > Modifier > Add Modifier > Boolean)
Set the operation: Union (fuse), Difference (cut), or Intersect
Pick the second object as the target
Apply the modifier, then delete the cutter object


Quick Reference
GoalMethodOne object, separate islandsCtrl+JFully merged verticesCtrl+J then Merge by DistanceObjects that overlap/intersectBoolean modifier → UnionWeld specific vertices onlyEdit Mode → select verts → M