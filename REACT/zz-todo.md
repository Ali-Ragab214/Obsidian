-react helmet
-react query (most important)
-react hook form
-(Auth)




ارجع لل LazyLoading اخر كام سطر 

```jsx 
export async function loader()
{
const data = await axios.get("url");

if(!data)
{
throw new Response('Failed to load products', { status: response.status });
}
return data.json();
}


export async function ErrorBoundary()
{
const error = useRouteError();
 return (
  <div> 
  <h2>حصل مشكلة!</h2> 
  <p>{error.message || 'Unknown error'}</p> </div> );
}


export default const  ProductPage()
{
const data = useLoderData();
}



in main.jsx 


{
path:'/dashboard',
lazy: asunc()=>
{
 const module = await import('./pages/Dashboard.jsx');
 return{
 Component:module.default,
 loader:module.default,
 ErorrBoundary:module.ErrorBoundary
 }
}
}

```