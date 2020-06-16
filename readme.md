![](https://i.imgur.com/ZB4uUaz.png)

# PluginMap GLTFJSX - Modified

Turns GLTF files into re-usable [react-three-fiber](https://github.com/react-spring/react-three-fiber) JSX components. 
Original Repo here: https://github.com/react-spring/gltfjsx/
See it in action here: https://twitter.com/0xca0a/status/1224335000755146753


## Modifications from Original
* Name Sensitization is OFF
* New Folder `builds` for Models and Generated JSX files
* Modified to Add Units, Floor, GreyArea
* Remove material if type == Unit
* Fix `nameExt` in `index.js` to have `file.replace(/^.*[\\\/]/, '')` 
* Install cli-color for proper messages


## Usage

```bash
 ## $ npx gltfjsx model.gltf [Model.js] [options]
 ## New Command 
 $ node .\index.js model.gltf [Model.js] [options]

 ## eg:/ $ node .\index.js .\builds\Ground_3D_1.1.1.glb .\builds\Ground_3D_1.1.1.js -c=false
```


### Options
```bash
  --draco, -d         Adds draco-Loader                   [boolean]
  --animation, -a     Extracts animation clips            [boolean]
  --types, -t         Adds Typescript definitions         [boolean]
  --compress, -c      Removes names and empty groups      [boolean] [default: true]
  --precision, -p     Number of fractional digits         [number ] [default: 2]
  --binary, -b        Draco binaries                      [string ] [default: '/draco-gltf/']
  --help              Show help                           [boolean]
  --version           Show version number                 [boolean]
```

You need to be set up for asset loading and the GLTF has to be present in your /public folder. This tools loads it, creates look-up tables of all the objects and materials inside, and writes out a JSX graph, which you can now alter comfortably. It will not change or alter your files in any way otherwise.

A typical result will look like this:

```jsx
// import * as THREE from 'three'
import React, { useRef } from 'react'
import { useLoader } from 'react-three-fiber'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'

import Logo from '../Logo'
import Unit from '../Unit'
import GrayArea from '../GrayArea'
import { useCountRenders } from '../../hooks/useCountRenders'

export default function Model(props) {
  const group = useRef()
  const { nodes, materials } = useLoader(GLTFLoader, 'models/Ground_3D_2.4.glb')

  useCountRenders('Map Model:' + props.name)

  return (
    <group ref={group} {...props} dispose={null}>
      <scene name="Scene">
        <Unit geometry={nodes.Unit_GA29.geometry} name="Unit_G.A29" position={[16.95, 0, -8.82]} />
        <Unit geometry={nodes.Unit_GA28.geometry} name="Unit_G.A28" position={[14.68, 0, -8.82]} />
        <Unit geometry={nodes.Unit_GA27.geometry} name="Unit_G.A27" position={[11.69, 0, -8.82]} />
        <Unit geometry={nodes.Unit_GA26.geometry} name="Unit_G.A26" position={[8.61, 0, -8.82]} />
        <Unit geometry={nodes.Unit_GA23a.geometry} name="Unit_G.A23a" position={[4.27, 0, -8.82]} />

        <mesh material={materials.Floor} geometry={nodes.Ground.geometry} name="Ground" position={[0, 0, 0]} />

        <mesh material={materials['Others.Normal']} geometry={nodes.Others_G4.geometry} name="Others_G4" position={[6.04, 0, -7.37]} />

        <GrayArea geometry={nodes.NoEntry_G1.geometry} name="NoEntry_G1" receiveShadow />

        <mesh material={materials['DalaHorse.Normal']} geometry={nodes.DalaHorse_G1.geometry} name="DalaHorse_G1" position={[-6.7, 0, -11.57]} rotation={[Math.PI / 2, 0, Math.PI]} />
		
		<group name="Icon_ATM_G_1" position={[-14.36, 1, -3.47]}>
          <mesh material={materials['IconWhite.Normal']} geometry={nodes['Curve.002_0'].geometry} name="Curve.002_0" />
          <mesh material={materials['IconBlue.Normal']} geometry={nodes['Curve.002_1'].geometry} name="Curve.002_1" />
        </group>

        <Logo material={materials['LcWaikiki.Normal']} geometry={nodes.Logo_LcWaikiki.geometry} name="Logo_LcWaikiki" position={[-8.87, 1, 6.27]} connectTo="Unit_G.05" />
        <Logo material={materials['BO.Normal']} geometry={nodes.Logo_BO.geometry} name="Logo_BO" position={[0.12, 1, 7.18]} connectTo="Unit_G.03a" />
      
      </scene>
    </group>
  )
}
```

This component is async and must be wrapped into `<Suspense>` for fallbacks:

```jsx
import React, { Suspense } from 'react'

function App() {
  return (
    <Suspense fallback={null}>
      <Model />
    </Suspense>
```

### --draco

Adds a DRACOLoader, for which you need to be set up. The necessary files have to exist in your /public folder. It defaults to `/draco-gltf/` which should contain [dracos gltf decoder](https://github.com/mrdoob/three.js/tree/dev/examples/js/libs/draco/gltf). It uses the draco shortcut from the [drei](https://github.com/react-spring/drei) library, which needs to be present in your package.json.

Your model will look like this:

```jsx
import { draco } from 'drei'

function Model(props) {
  const { nodes, materials } = useLoader(GLTFLoader, '/model.gltf', draco('/draco-gltf/'))
```

### --animation

If your GLTF contains animations it will add a THREE.AnimationMixer to your component and extract the clips:

```jsx
const actions = useRef()
const [mixer] = useState(() => new THREE.AnimationMixer())
useFrame((state, delta) => mixer.update(delta))
useEffect(() => {
  actions.current = { storkFly_B_: mixer.clipAction(gltf.animations[0]) }
  return () => gltf.animations.forEach((clip) => mixer.uncacheClip(clip))
}, [])
```

If you want to play an animation you can do so at any time:

```jsx
<mesh onClick={(e) => actions.current.storkFly_B_.play()} />
```

### --types

This will make your GLTF typesafe.

```tsx
type GLTFResult = GLTF & {
  nodes: {
    cube1: THREE.Mesh
    cube2: THREE.Mesh
  }
  materials: {
    base: THREE.MeshStandardMaterial
    inner: THREE.MeshStandardMaterial
  }
}

function Model(props: JSX.IntrinsicElements['group']) {
  const { nodes, materials } = useLoader<GLTFResult>(GLTFLoader, '/model.gltf')
```

## Release notes and breaking changes

https://github.com/react-spring/gltfjsx/blob/master/whatsnew.md
