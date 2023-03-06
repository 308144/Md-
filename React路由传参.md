React路由传参

```ini
import {BrowserRouter,Route,Routes} from "react-router-dom"


function  App(){
	<BrowserRouter>
		<Routes>
		<Route path="/" element={<Home/>}/>
		      <Route path="/dashboard/:id/:data" element={<Dashboard />} />
		</Routes>
	</BrowserRouter>
}

<Link to="/dashboard/:id/:data"   />


```

