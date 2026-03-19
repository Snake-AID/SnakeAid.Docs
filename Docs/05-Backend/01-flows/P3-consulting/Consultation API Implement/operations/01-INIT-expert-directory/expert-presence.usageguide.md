---
doc_role: operation-specific
module: consultation
operation: 01-INIT-expert-directory
feature: expert-presence
kind: usage-guide
status: active
last_updated: 2026-03-19
owners: [backend-team, mobile-team]
---

# Expert Presence Management Usage Guide

## Mục tiêu tài liệu

Tài liệu này hướng dẫn mobile dev tích hợp **expert online/offline presence** - tính năng cho phép users xem và theo dõi trạng thái online của experts trong thời gian thực.

## Tổng quan kiến trúc

### Components chính

- **ExpertProfile.IsOnline**: Boolean field trong database
- **ExpertHub**: SignalR hub quản lý presence connections
- **SignalRExpertEmergencyNotificationService**: Service track expert connections
- **ExpertProfileResponse.isOnline**: API field trả về status

### Flow hoạt động

1. **Expert app** connect ExpertHub → gọi `JoinAsExpert()` → set `IsOnline = true`
2. **User app** connect ExpertHub → gọi `JoinAsMember()` → nhận `OnlineExpertsSnapshot`
3. **Real-time updates**: `ExpertPresenceChanged` event khi expert online/offline
4. **Disconnect**: Expert disconnect → auto set `IsOnline = false`

## API Endpoints

### GET /api/experts - List Experts với Online Status

**Query Parameters**
```javascript
{
  pageNumber: 1,
  pageSize: 20,
  specialization?: "Venomous Snakes",
  isOnline?: true,        // Filter chỉ experts online
  sortBy?: "isOnline"     // Sort theo online status
}
```

**Response Sample**
```json
{
  "status_code": 200,
  "data": {
    "items": [
      {
        "accountId": "expert-uuid",
        "name": "Dr. John Doe",
        "isOnline": true,      // ← Expert online status
        "consultationFee": 150000,
        "rating": 4.8,
        // ... other fields
      }
    ]
  }
}
```

### GET /api/experts/{expertId} - Expert Detail

**Response includes**
```json
{
  "isOnline": true,  // Current online status
  "name": "Dr. John Doe",
  // ... other profile data
}
```

## SignalR Integration

### Hub Connection
```javascript
const connection = new signalR.HubConnectionBuilder()
  .withUrl('/hubs/expert')
  .build();
```

### Expert App Methods

#### JoinAsExpert() - Set Online
```javascript
await connection.invoke('JoinAsExpert');
```

**Effects:**
- Sets `ExpertProfile.IsOnline = true` in database
- Registers connection in presence tracking service
- Broadcasts `ExpertPresenceChanged` to all users

**Response:**
```json
{
  "ExpertId": "uuid",
  "ConnectionId": "signalr-conn-id",
  "Message": "Expert connected successfully."
}
```

#### Auto Disconnect Handling
```javascript
connection.onclose(async () => {
  // Expert automatically set offline
  // No manual intervention needed
});
```

### User App Methods

#### JoinAsMember() - Subscribe to Presence
```javascript
await connection.invoke('JoinAsMember');
```

**Effects:**
- Adds to "ConsultationMembers" group
- Receives initial `OnlineExpertsSnapshot`

## SignalR Events

### OnlineExpertsSnapshot - Initial Online List
```javascript
connection.on('OnlineExpertsSnapshot', (data) => {
  console.log('Current online experts:', data.onlineExpertIds);
  // Update UI with initial online status
});
```

**Payload:**
```json
{
  "onlineExpertIds": [
    "expert-uuid-1",
    "expert-uuid-2"
  ],
  "serverTimeUtc": "2026-03-19T04:30:00Z"
}
```

### ExpertPresenceChanged - Real-time Updates
```javascript
connection.on('ExpertPresenceChanged', (data) => {
  const { expertId, isOnline, changedAtUtc } = data;
  // Update expert online status in UI
  updateExpertOnlineStatus(expertId, isOnline);
});
```

**Payload:**
```json
{
  "expertId": "expert-uuid",
  "isOnline": true,  // true = online, false = offline
  "changedAtUtc": "2026-03-19T04:31:00Z"
}
```

## Mobile Integration Patterns

### React Native Example
```typescript
import { useSignalR } from './hooks/useSignalR';

function ExpertDirectoryScreen() {
  const [experts, setExperts] = useState([]);
  const [onlineExperts, setOnlineExperts] = useState(new Set());

  // Connect to SignalR
  useSignalR('/hubs/expert', {
    onConnected: async (connection) => {
      await connection.invoke('JoinAsMember');
    },

    events: {
      OnlineExpertsSnapshot: (data) => {
        setOnlineExperts(new Set(data.onlineExpertIds));
      },

      ExpertPresenceChanged: (data) => {
        setOnlineExperts(prev => {
          const updated = new Set(prev);
          if (data.isOnline) {
            updated.add(data.expertId);
          } else {
            updated.delete(data.expertId);
          }
          return updated;
        });
      }
    }
  });

  // Fetch experts list
  useEffect(() => {
    fetchExperts();
  }, []);

  const fetchExperts = async () => {
    const response = await fetch('/api/experts?isOnline=true');
    const data = await response.json();
    setExperts(data.data.items);
  };

  return (
    <FlatList
      data={experts}
      renderItem={({ item }) => (
        <ExpertCard
          expert={item}
          isOnline={onlineExperts.has(item.accountId)}
        />
      )}
    />
  );
}
```

### Flutter Example
```dart
class ExpertDirectoryBloc extends Bloc<ExpertDirectoryEvent, ExpertDirectoryState> {
  final SignalRService _signalRService;

  ExpertDirectoryBloc(this._signalRService) : super(ExpertDirectoryInitial()) {
    // Connect to SignalR
    _signalRService.connect('/hubs/expert');

    // Join as member
    _signalRService.invoke('JoinAsMember');

    // Listen to presence events
    _signalRService.on('OnlineExpertsSnapshot', (data) {
      final onlineIds = Set<String>.from(data['onlineExpertIds']);
      add(OnlineExpertsUpdated(onlineIds));
    });

    _signalRService.on('ExpertPresenceChanged', (data) {
      final expertId = data['expertId'] as String;
      final isOnline = data['isOnline'] as bool;
      add(ExpertPresenceChanged(expertId, isOnline));
    });
  }

  Future<void> _onLoadExperts(LoadExperts event, Emitter emit) async {
    final response = await http.get('/api/experts');
    final experts = ExpertList.fromJson(json.decode(response.body));
    emit(ExpertsLoaded(experts));
  }

  Future<void> _onOnlineExpertsUpdated(OnlineExpertsUpdated event, Emitter emit) async {
    // Update online status overlay
    emit(state.copyWith(onlineExpertIds: event.onlineIds));
  }

  Future<void> _onExpertPresenceChanged(ExpertPresenceChanged event, Emitter emit) async {
    final updatedOnlineIds = Set<String>.from(state.onlineExpertIds);
    if (event.isOnline) {
      updatedOnlineIds.add(event.expertId);
    } else {
      updatedOnlineIds.remove(event.expertId);
    }
    emit(state.copyWith(onlineExpertIds: updatedOnlineIds));
  }
}
```

## Database Schema

### ExpertProfile Table
```sql
CREATE TABLE ExpertProfile (
  AccountId UNIQUEIDENTIFIER PRIMARY KEY,
  Biography NVARCHAR(2000),
  IsOnline BIT NOT NULL DEFAULT 0,  -- ← Presence field
  ConsultationFee DECIMAL(18,2),
  -- ... other columns
);
```

### Presence Tracking Logic
- **Connect**: `UPDATE ExpertProfile SET IsOnline = 1 WHERE AccountId = @expertId`
- **Disconnect**: `UPDATE ExpertProfile SET IsOnline = 0 WHERE AccountId = @expertId`
- **Query**: `SELECT IsOnline FROM ExpertProfile WHERE AccountId = @expertId`

## Error Handling

### Connection Issues
```javascript
connection.onclose(() => {
  // Connection lost - show offline indicators
  // Retry connection logic
});

connection.onerror((error) => {
  console.error('SignalR error:', error);
  // Handle connection errors
});
```

### API Errors
```javascript
try {
  const response = await fetch('/api/experts');
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  const data = await response.json();
  // Process data
} catch (error) {
  // Handle network/API errors
  // Show offline mode UI
}
```

## Performance Considerations

### Connection Management
- **Single connection** per user app session
- **Auto-reconnect** on disconnect
- **Connection pooling** for multiple tabs

### State Synchronization
- **Initial load**: REST API for current state
- **Real-time sync**: SignalR for live updates
- **Conflict resolution**: SignalR takes precedence

### Memory Management
- **Cleanup listeners** on component unmount
- **Debounce updates** to prevent UI thrashing
- **Pagination** for large expert lists

## Testing Strategy

### Unit Tests
```typescript
describe('ExpertPresenceService', () => {
  it('should update online status on connect', async () => {
    const service = new ExpertPresenceService();
    await service.setOnlineStatus(expertId, true);
    expect(database.isOnline(expertId)).toBe(true);
  });

  it('should broadcast presence changes', async () => {
    const mockHub = { broadcast: jest.fn() };
    await service.broadcastPresenceChange(expertId, true);
    expect(mockHub.broadcast).toHaveBeenCalledWith('ExpertPresenceChanged', {
      expertId,
      isOnline: true
    });
  });
});
```

### Integration Tests
```typescript
describe('Expert Directory E2E', () => {
  it('should show online experts in real-time', async () => {
    // Connect user app
    await connectUserApp();

    // Expert goes online
    await expertApp.joinAsExpert();

    // Verify user sees online status
    await waitFor(() => {
      expect(screen.getByText('Online')).toBeVisible();
    });

    // Expert disconnects
    await expertApp.disconnect();

    // Verify user sees offline status
    await waitFor(() => {
      expect(screen.getByText('Offline')).toBeVisible();
    });
  });
});
```

## Deployment Checklist

- [ ] ExpertHub registered in Program.cs
- [ ] SignalR CORS configured for mobile apps
- [ ] Database migration includes IsOnline column
- [ ] ExpertProfileResponse includes isOnline field
- [ ] Mobile apps updated to handle presence events
- [ ] Error handling for connection failures
- [ ] Performance monitoring for SignalR connections

## Troubleshooting

### Common Issues

**Experts not showing online:**
- Check ExpertHub connection
- Verify `JoinAsExpert()` called
- Check database IsOnline field

**Presence updates not received:**
- Verify user called `JoinAsMember()`
- Check SignalR connection status
- Confirm CORS configuration

**Stale online status:**
- Check for disconnect handling
- Verify auto-cleanup on server restart
- Monitor connection timeouts

## Related Documentation

- `consultation.usageguide.md` - Full consultation API reference
- `AGENTS-USAGEGUIDE.md` - Agent integration patterns
- SignalR documentation for mobile platforms
