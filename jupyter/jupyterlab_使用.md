1. 通过url可以管理workspaces

   ## Cloning Workspaces

   You can copy the contents of a workspace into another workspace with the `clone` url parameter.

   To copy the contents of the workspace `foo` into the workspace `bar`:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/bar?clone=foo
   ```

   To copy the contents of the default workspace into the workspace `foo`:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/foo?clone
   ```

   To copy the contents of the workspace `foo` into the default workspace:

   ```
   http(s)://<server:port>/<lab-location>/lab?clone=foo
   ```

   ## Resetting a Workspace

   Use the `reset` url parameter to clear a workspace of its contents.

   To reset the contents of the workspace `foo`:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/foo?reset
   ```

   To reset the contents of the default workspace:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/lab?reset
   ```

   ## Combining URL Functions

   These URL functions can be used separately, as above, or in combination.

   To reset the workspace `foo` and load a specific notebook afterward:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/foo/tree/path/to/notebook.ipynb?reset
   ```

   To clone the contents of the workspace `bar` into the workspace `foo` and load a notebook afterward:

   ```
   http(s)://<server:port>/<lab-location>/lab/workspaces/foo/tree/path/to/notebook.ipynb?clone=bar
   ```

   To reset the contents of the default workspace and load a notebook:

   ```
   http(s)://<server:port>/<lab-location>/lab/tree/path/to/notebook.ipynb?reset
   ```



2. 右键一个文件, 设置"Copy Shareable Link"可以在下次访问时, 直接以文件打开形式运行
3. 可以给每一个notebook设置一个console, 右键notebook, 选择"New Console for Notebook"

4. shift+tab可以显示一个额外提示窗口