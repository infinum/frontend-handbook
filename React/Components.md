## ./src/components/ui/TripList.tsx

- no logic should happen here outside of data presentation
- they should be as simple as possible
- you **should** place **input handler/useForm** here
- they should allow composition of other components
- "local" styled components can be here, but extract a-priori

```tsx
import React from 'react';
const styled from 'emotion';

const Main = styled.div``;

/**
 * Links to Invision/...
 */
const TripList = ({ trips, children }) => {
	return (
		<Main>
			{children(trips)}
		</Main>
	);
};
```